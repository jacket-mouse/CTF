# base编解码

```python
import base64

encoded_str = "PD9waHANCiRmbGFnPSJjeWJlcnBlYWNlezc5Yzk3ZWEyYTdmOTNiZTdiYjc3YmNmMDE4NjE1ODhifSI7DQo/Pg=="
decoded_bytes = base64.b64decode(encoded_str)

decoded_str = decoded_bytes.decode("utf-8")

print(decoded_str)


```

