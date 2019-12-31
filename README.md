# 概述

一款基于 js-cookie、axios ，通用微信网页授权的 js 插件，以获取 openid unionid 等用户信息

# 配置

| 属性      | 说明                                                                          | 类型     | 必填 | 默认值      |
| --------- | ----------------------------------------------------------------------------- | -------- | ---- | ----------- |
| appId     | 公众号的唯一标识                                                              | String   | 是   | -           |
| scope     | 应用授权作用域 `snsapi_base` `snsapi_userinfo`，详情请查看微信开发文档        | String   | 否   | snsapi_base |
| expires   | Cookies 过期时间/天                                                           | Number   | 否   | 30          |
| state     | 重定向后会带上 `state` 参数，开发者可以填写 a-zA-Z0-9 的参数值，最多 128 字节 | String   | 否   | -           |
| oauthUrl  | 服务器授权，绝对地址，获取 `openid` `unionid`                                 | String   | 是   | -           |
| onSuccess | 服务器授权，成功回调，需要返回 `openId` `unionId` `userInfo` 等数据做逻辑处理 | Function | 是   | -           |
| onFail    | 服务器授权，失败回调                                                          | Function | 否   | -           |

# 方法

| 方法名      |           说明           | 参数 |
| ----------- | :----------------------: | :--: |
| init        |          初始化          |  -   |
| oauth       | 授权，再调调用可切换账号 |  -   |
| getUserInfo |       获取用户信息       |  -   |

# 示例

```javascript
import WxOauth from "@/utils/wx/oauth";

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

# 作者

SuperIron
