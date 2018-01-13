# passport-enterprise-wechat
[passportjs](http://passportjs.org/) strategy for authenticating with [Enterprise Wechat Accounts](https://qy.weixin.qq.com/)
using the OAuth 2.0 API.

passport 企业微信OAuth2.0, 网页授权登录用户验证模块,[企业微信网页授权登录API文档](https://work.weixin.qq.com/api/doc#10028)。

代码改写基于 [passport-wechat-work](https://github.com/harryhan1989/passport-wechat-work#readme) 企业微信扫码授权登。

微信企业号, 参看 [passport-wechat-enterprise](https://github.com/wenwei1202/passport-wechat-enterprise)

微信公众号, 参看 [passport-wechat-public](https://github.com/wenwei1202/passport-wechat-public)

## Install

```
$ npm install passport-enterprise-wechat
```

## Usage

### Configure passport

```
var passport = require('passport');
app.use(passport.initialize());
app.use(passport.session());
```

#### Configure Strategy

- 在 passport 注册 EnterpriseWechatStrategy , passport.use() 的第一个参数是策略名称 , `enterprise-wechat`。EnterpriseWechatStrategy 的构造函数的参数是 options , verify 以及 getAccessToken 和 saveAccessToken。

- options 的 corpId, corpSecret, agentId 和 callbackURL 是必填项, 其他项可选填。

- 认证成功后 verify 函数将用户信息通过 profile 传给 done 函数 , profile 对象包含 `UserId` , `DeviceId` , `errcode` , `errmsg` ,  `id` 等 , 返回的信息根据 `scope` 配置的参数会有所不同。

- getAccessToken 和 saveAccessToken 用于获得 AccessToken 和保存新的 AccessToken , 当 getAccessToken 返回的 AccessToken 无效时 , 会通过调用微信的 `/gettoken` 接口获取新的 AccessToken , 并用 saveAccessToken 进行保存。
getAccessToken 和 saveAccessToken 都是必需的。

- saveAccessToken 返回的是一个 AccessToken 对象 , 对象包含接口凭证 `access_token` , 接口有效时间 `expires_in` , 以及  token 获取的时间 `create_at`。

例如：在 [Express](http://expressjs.com/) 程序 `app.js` 文件中添加配置

```
var EnterpriseWechatStrategy = require('passport-enterprise-wechat').Strategy;

passport.use("enterprise-wechat", new EnterpriseWechatStrategy({
  corpId: CORP_ID,
  corpSecret: CORP_SECRET,
  agentId: AGENT_ID,
  callbackURL: "http://localhost:3000/auth/enterpriseWechatLogin/callback",
  state: "state",
  scope: "snsapi_base",
},
  function (profile, done) {
    done(null, profile);
  },
  function getAccessToken(callback) {
    callback();
  },
  function saveAccessToken(accessToken) {
    console.log('accessToken', accessToken);
  }
));

passport.serializeUser(function (user, done) {
  console.log('user', user);
  done(null, user);
});
```

#### Authenticate Requests

用 `passport.authenticate()` 在对应的 route 下 , 注意 strategy 名字和 passport.use() 时一致。

例如：在 [Express](http://expressjs.com/) 程序路由中配置

```
app.get('/auth/enterpriseWechatLogin',
  passport.authenticate('enterprise-wechat'));

app.get('/auth/enterpriseWechatLogin/callback',
  passport.authenticate('enterprise-wechat',
    {
      failureRedirect: '/authFailure',
    }),
  function (req, res) {
    res.redirect('/');
  });
```
