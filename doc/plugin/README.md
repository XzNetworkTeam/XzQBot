# XzBot SDK

- [使用规范](#使用规范)
- [参数](#参数)
- [响应](#响应)
- [异步调用](#异步调用)
- [限速调用](#限速调用)
- [相关配置](#相关配置)

XzBot SDK 是 XzQBot 向用户提供的开发工具包，用户可通过已实例化好的类直接调用相应的api函数。详见[XzBot SDK API](SDK.md)。

## 使用规范

框架在**初始化**和**重载**插件时会调用 插件的 `initialize` 函数，即**初始化函数**，函数的第一个形参即是已实例化好的 XzBot 对象 —— `bot`，插件需自行存储 `bot` 对象。

**框架处理插件流程：**

- 初始化：
  - 1.校验插件有效性
    - 失败 => 取消加载，并不会加入到插件实例组中（无法在插件管理中看到）
  - 2.获取插件基础信息
  - 3.同步调用插件 `initialize` 函数，并传入已实例化的 `bot` 对象
- 收到主框架事件：
  - 同步调用 `onEvent` 函数，并传入 event 对象
- 销毁：
  - 同步调用插件 `destroy` 函数

*注意：框架调用 `initialize`、`onEvent`、`destroy` 函数时均是同步调用，当您的函数是异步函数时框架不会等待调用完成。*

```javascript
var bot = null;
const XzBot = require("../api/bot").default;
const quickReply = require("../constants/quickReply").default;

module.exports = {
    info: {
        name: 'Example Plugin', // 插件名
        version: '1.0.0', // 插件版本
        description: 'This is an example plugin.', // 插件描述
        author: 'your name', // 插件作者
        uuid: '$uuid$' // 插件唯一标识
    },
    initialize: function(_bot) {
        // 初始化插件
        bot = _bot;
    },

    destroy: function() {
        // 销毁插件，清理定时器，ws连接等操作
    },
    onEvent: function(event) {
        // 数据处理
        switch (event.post_type) {
            case 'message':
                const IS_GROUP = event.message_type === 'group';
                const IS_PRIVATE = event.message_type === 'private';
                if (IS_GROUP || IS_PRIVATE) {
                    // 通用类型处理器
                    let gid = event.group_id;
                    let qid = event.user_id;
                    let raw_message = event.raw_message;
                    let seg_message = event.message;
                    let message_id = event.message_id;

                    // if (IS_GROUP) bot.sendGroupMsg(gid, [XzBot.reply(message_id), res]);
                    // if (IS_PRIVATE) bot.sendPrivateMsg(qid, res);
                }
                if (IS_GROUP) {
                    let gid = event.group_id;
                    let qid = event.user_id;
                    let raw_message = event.raw_message;
                    let seg_message = event.message;
                    let message_id = event.message_id;
                    let sender_name = event.sender.nickname;
                    let sender_card = event.sender.card || message.sender.nickname;
                    
                    // 处理群聊消息
                } else if (IS_PRIVATE) {
                    let qid = event.user_id;
                    let raw_message = event.raw_message;
                    let seg_message = event.message;
                    let message_id = event.message_id;
                    
                    // 处理私聊消息
                }
                break;
            default:
                break;
        }
    }
};
```

## 参数

API 函数 调用需要指定所需的参数。

在后面的 API 函数 描述中，函数名 在标题中给出，如 `sendPrivateMsg`；参数在「参数」小标题下给出，其中「数据类型」使用 JSON 中的名字，例如 `string`、`number` 等。

特别地，数据类型 `message` 表示该参数是一个消息类型的参数，在调用 API 时，`message` 类型的参数允许接受字符串、消息段数组、单个消息段对象三种类型的数据，关于消息格式的更多细节请查看 [消息](../message/)。

## 响应

XzBot 会对每个 API 调用返回一个 JSON 响应（除非是 HTTP 状态码不为 200 的情况），响应中的 `data` 字段包含 API 调用返回的数据内容。在后面的 API 描述中，将只给出 `data` 字段的内容，放在「响应数据」小标题下，而不再赘述 `status`、`retcode` 字段。

## 异步调用

所有 API 几乎都是异步函数，都将返回一个 Promise。

## 限速调用

XzBot SDK 提供一个 `setRateLimited` 函数来开启或关闭排队模式。

主要是用在发送消息接口上，以避免消息频率过快导致腾讯封号。所有限速调用将会以指定速度**排队执行**，这个速度可在配置中指定。

限速调用的响应中，`status` 字段为 `async`。

## 相关配置

| 配置项 | 默认值 | 说明 |
| -------- | ------ | --- |
| `api.rate_limit_interval` | `500` | 限速 API 调用的排队间隔时间，单位毫秒 |

<hr>

| 上一节 | 下一节 |
| --- | --- |
| [消息段类型](../message/segment.md) | [XzBot SDK API](SDK.md) |
