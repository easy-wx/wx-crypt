# wx-crypt

企业微信 / 公众号消息加解密工具，支持企业微信 XML 回调协议、公众号回调协议，以及企业微信智能机器人（AI Bot）的 JSON 回调协议。

## 安装

```bash
pip install wx-crypt
```

## 支持的通道

```python
from wx_crypt import (
    WXBizMsgCrypt,
    WxChannel_Wecom,
    WxChannel_Mp,
    WxChannel_WecomAiBot,
)
```

| 通道 | 说明 | 回包格式 |
| --- | --- | --- |
| `WxChannel_Wecom` | 企业微信 XML 协议，默认值 | XML |
| `WxChannel_Mp` | 公众号协议 | XML |
| `WxChannel_WecomAiBot` | 企业微信智能机器人（AI Bot）JSON 协议 | JSON |

不传 `channel` 时默认使用 `WxChannel_Wecom`，因此旧的企业微信 XML 处理方式保持兼容。

## 基础用法

```python
from wx_crypt import WXBizMsgCrypt, WxChannel_Wecom

crypt = WXBizMsgCrypt(
    sToken="your_token",
    sEncodingAESKey="your_encoding_aes_key",
    sReceiveId="your_receive_id",
    channel=WxChannel_Wecom,
)

ret, plain_text = crypt.DecryptMsg(
    sPostData=request_body,
    sMsgSignature=msg_signature,
    sTimeStamp=timestamp,
    sNonce=nonce,
)

ret, response_body = crypt.EncryptMsg(
    sReplyMsg=reply_xml,
    sNonce=nonce,
    timestamp=timestamp,
)
```

## 企业微信智能机器人（AI Bot）

`WxChannel_WecomAiBot` 用于企业微信智能机器人的 JSON 回调协议。该通道复用原有 AES-CBC 加解密和 SHA1 签名逻辑，只把消息载体从 XML 切换为 JSON。

```python
from wx_crypt import WXBizMsgCrypt, WxChannel_WecomAiBot

crypt = WXBizMsgCrypt(
    sToken="your_token",
    sEncodingAESKey="your_encoding_aes_key",
    sReceiveId="your_bot_id_or_receive_id",
    channel=WxChannel_WecomAiBot,
)

ret, plain_text = crypt.DecryptMsg(
    sPostData=request_body,      # JSON body: {"encrypt": "..."}
    sMsgSignature=msg_signature,
    sTimeStamp=timestamp,
    sNonce=nonce,
)

reply_json = '{"msgtype":"text","text":{"content":"hello"}}'
ret, response_body = crypt.EncryptMsg(
    sReplyMsg=reply_json.encode("utf-8"),
    sNonce=nonce,
    timestamp=timestamp,
)
```

AI Bot 的加密回包是 JSON 字符串，格式如下：

```json
{
  "encrypt": "msg_encrypt",
  "msgsignature": "msg_signature",
  "timestamp": 1641002400,
  "nonce": "nonce"
}
```

## 兼容性说明

- `WxChannel_Wecom` 和 `WxChannel_Mp` 仍然使用 XML 解析和 XML 回包。
- 只有显式传入 `channel=WxChannel_WecomAiBot` 时，才会使用 JSON body 解析和 JSON 回包。
- `DecryptMsg` / `EncryptMsg` 的返回形式保持为 `(ret, data)`；`ret == 0` 表示成功。
