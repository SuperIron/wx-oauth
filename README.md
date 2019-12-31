# 概述

一款基于 js-cookie、axios ，通用微信网页授权的 js 插件，以获取 openid unionid 等用户信息

# 配置

| 属性      | 说明                                                                        | 类型     | 必填 | 默认值      |
| --------- | --------------------------------------------------------------------------- | -------- | ---- | ----------- |
| appId     | 公众号的唯一标识                                                            | String   | 是   | -           |
| scope     | 应用授权作用域 `snsapi_base` `snsapi_userinfo`，详情请查看微信开发文档         | String   | 否   | snsapi_base |
| expires   | Cookies 过期时间/天                                                         | Number   | 否   | 30          |
| state     | 重定向后会带上 `state` 参数，开发者可以填写 a-zA-Z0-9 的参数值，最多 128 字节 | String   | 否   | -           |
| oauthUrl  | 服务器授权，绝对地址，获取 `openid` `unionid`                                   | String   | 是   | -           |
| onSuccess | 服务器授权，成功回调，需要返回 `openId` `unionId` `userInfo` 等数据做逻辑处理     | Function | 是   | -           |
| onFail    | 服务器授权，失败回调                                                        | Function | 否   | -           |

# 方法

| 方法名      |           说明           | 参数 |
| ----------- | :----------------------: | :--: |
| init        |          初始化          |  -   |
| oauth       | 授权，再调调用可切换账号 |  -   |
| getUserInfo |       获取用户信息       |  -   |

# 示例

```javascript
import WxOauth from "@/utils/wx/oauth.class";

const wxOauth = new WxOauth({
    appId: "your appId",
    oauthUrl: "your oauthUrl",
    onSuccess: res => {
        // 处理成功回调后，获取unionId openId userInfo等信息并返回

        const unionId = "unionId";
        const openId = "openId";
        const userInfo = "userInfo";

        return {
            unionId,
            openId,
            userInfo
        };
    }
});

wxOauth.init();
```

# 代码

```javascript
import Cookies from "js-cookie";
import axios from "axios";

class WxOauth {
    /**
     * @param appId         公众号的唯一标识
     * @param scope         应用授权作用域snsapi_base|snsapi_userinfo，默认snsapi_base
     * @param expires       Cookies过期时间/天，默认30天
     * @param state         重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节
     * @param oauthUrl      服务器授权，绝对地址，获取openid unionid
     * @param onSuccess     服务器授权，成功返回
     * @param onFail        服务器授权，失败返回
     */
    constructor(options) {
        const {
            appId,
            scope,
            expires,
            state,
            oauthUrl,
            onSuccess,
            onFail
        } = options;

        this.appId = appId;
        this.scope = scope || "snsapi_base";
        this.isSnsapiBase = scope === "snsapi_base";
        this.expires = expires || 30;
        this.state = state;
        this.oauthUrl = oauthUrl;

        this.onSuccess = onSuccess;
        this.onFail = onFail || function() {};
    }

    /**
     * @description `判断数据是否为对象`
     *
     * @param {*} data
     */
    _isObject(data) {
        return Object.prototype.toString.call(data) === "[object Object]";
    }

    /**
     * @description 获取请求参数，支持解析带#的url
     *
     * @param {name} name 参数名
     */
    _getQueryString(name) {
        return (
            decodeURIComponent(
                (new RegExp("[?|&]" + name + "=" + "([^&;]+?)(&|#|;|$)").exec(
                    location.href
                ) || ["", ""])[1].replace(/\+/g, "%20")
            ) || null
        );
    }

    /**
     * @description 获取拼接后的url，过滤undefined|null数据
     *
     * @param {String} url
     * @param {Object} data
     */
    _getUrl(url, data) {
        if (!this._isObject(data)) {
            return url;
        }

        const params = Object.keys(data).reduce((pre, key) => {
            const value = data[key];

            value !== undefined &&
                value !== null &&
                (pre += `&${key}=${value}`);
            return pre;
        }, "");

        return `${url}${params.replace("&", "?")}`;
    }

    /**
     * @description 服务器授权，获取用户信息
     */
    _oauth(code, state) {
        return new Promise((resolve, reject) => {
            const options = {
                method: "POST",
                data: {
                    code,
                    state
                },
                url: this.oauthUrl
            };

            axios
                .request(options)
                .then(res => {
                    const { openId, unionId, userInfo } = this.onSuccess(
                        res.data
                    );

                    Cookies.set(this._getCookieName("unionId"), unionId, {
                        expires: this.expires
                    });
                    Cookies.set(this._getCookieName("openId"), openId, {
                        expires: this.expires
                    });
                    Cookies.set(this._getCookieName("userInfo"), userInfo, {
                        expires: this.expires
                    });

                    resolve();
                })
                .catch(err => this.onFail(err));
        });
    }

    /**
     * @description 生成特定cookie名
     */
    _getCookieName(name) {
        return `wx_oauth_${this.appId}_${name}`;
    }

    /**
     * @description 是否已登录
     */
    _isLogged(openId, unionId) {
        return this.isSnsapiBase ? openId : unionId || openId;
    }

    /**
     * @description 登录，通过code获取用户信息
     */
    async _login() {
        const code = this._getQueryString("code");

        if (code) {
            const state = this._getQueryString("state");

            await this._oauth(code, state);
            return;
        }

        this.oauth();
    }

    init() {
        const unionId = Cookies.get(this._getCookieName("unionId"));
        const openId = Cookies.get(this._getCookieName("openId"));

        !this._isLogged(openId, unionId) && this._login();
    }

    /**
     * @description 微信授权，支持切换账号
     */
    oauth() {
        Cookies.remove(this._getCookieName("unionId"));
        Cookies.remove(this._getCookieName("openId"));

        const url = this._getUrl(
            "https://open.weixin.qq.com/connect/oauth2/authorize",
            {
                appid: this.appId,
                redirect_uri: encodeURIComponent(location.href),
                response_type: "code",
                scope: this.scope,
                state: this.state
            }
        );
        location.href = `${url}#wechat_redirect`;
    }

    /**
     * @description 获取用户信息
     */
    getUserInfo() {
        const userInfo = Cookies.get(this._getCookieName("userInfo"));

        return userInfo ? JSON.parse(userInfo) : {};
    }
}

export default WxOauth;
```

# 作者

SuperIron
