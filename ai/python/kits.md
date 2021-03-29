# kit-python小技巧

## Group A

### 字符串 和 python字典

```python
import json
a = ''' {
      "num": 654321,
      "numId": null,
      "Name": "简书",
      "netId": null,
      "Shorthand": null,
      "bonus": 0
    }
    '''
str_json = json.loads(a)
print(str_json)
```

---

python字典获取元素