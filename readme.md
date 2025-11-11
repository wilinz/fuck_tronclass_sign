# fuck_tronclass_sign 畅课系统签到url解析

源 js：
```js
 , fe = function(e) {
                    if (t.log("scanUrlAnalysis url:".concat(e)),
                    e.includes("/j?p=") && !e.startsWith("http") && (e = de() + e),
                    !e.startsWith("http"))
                        return e;
                    var n;
                    try {
                        n = new URL(e)
                    } catch (ae) {
                        return e
                    }
                    if ("/j" === n.pathname || "/scanner-jumper" === n.pathname) {
                        var o = new URLSearchParams(n.search)
                          , r = null;
                        try {
                            var a = o.get("_p");
                            a && (r = JSON.parse(a))
                        } catch (q) {}
                        return r || (r = (0,
                        G.V)(o.get("p") || "")),
                        r && Object.keys(r).length ? JSON.stringify(r) : e
                    }
                    return e
                }
```
```js
 var ta = String.fromCharCode(30)
          , ea = String.fromCharCode(31)
          , na = String.fromCharCode(26)
          , ra = String.fromCharCode(16)
          , ia = na + "1"
          , oa = na + "0"
          , aa = Object.fromEntries(["courseId", "activityId", "activityType", "data", "rollcallId", "groupSetId", "accessCode", "action", "enableGroupRollcall", "createUser", "joinCourse"].map((function(t, e) {
            return [t, e.toString(36)]
        }
        )))
          , ua = Object.fromEntries(["classroom-exam", "feedback", "vote"].map((function(t, e) {
            return [t, na + Number(e + 2).toString(36)]
        }
        )))
          , ca = Object.fromEntries(Object.entries(aa).map((function(t) {
            return [t[1], t[0]]
        }
        )))
          , sa = Object.fromEntries(Object.entries(ua).map((function(t) {
            return [t[1], t[0]]
        }
        )))
          , la = function(t) {
            var e = {};
            return t && "string" == typeof t ? (t.split("!").filter((function(t) {
                return !!t
            }
            )).forEach((function(t) {
                var n = t.split("~")
                  , r = n[0]
                  , i = n[1];
                r && void 0 !== i && (e[function(t) {
                    return ca[t] || t
                }(r)] = function(t) {
                    if (t.startsWith(na))
                        return t === ia || t !== oa && (sa[t] || t);
                    if (t.startsWith(ra)) {
                        var e = t.substring(1).split(".").map((function(t) {
                            return parseInt(t, 36)
                        }
                        ));
                        return e.length > 1 ? parseFloat(e[0] + "." + e[1]) : e[0]
                    }
                    return t.replaceAll(ea, "~").replaceAll(ta, "!")
                }(i))
            }
            )),
            e) : e
        }
    }
```

貌似这是整个请求体：
```js
if (t.rollcallId && e.latitude) {
                    var n = new u.YY(G());
                    n.setCourse(d["default"].state.course.currentCourse),
                    n.rollcallId = t.rollcallId,
                    n.location = {
                        lat: e.latitude,
                        lon: e.longitude
                    },
                    n.accuracy = e.accuracy,
                    n.altitude = e.altitude,
                    n.altitudeAccuracy = e.altitudeAccuracy,
                    n.heading = e.heading,
                    n.speed = e.speed,
                    n.actionType = t.actionType,
                    n.deviceId = t.deviceId,
                    n.studentStatus = t.studentStatus,
                    n.studentDistance = t.studentDistance,
                    s.cleanObjectNil(n),
                    z(n)
                }
```

算法还原
```python
# 定义特殊字符
ta = chr(30)
ea = chr(31)
na = chr(26)
ra = chr(16)
ia = na + "1"
oa = na + "0"

# 辅助函数：将整数转换为 base36 字符串
def to_base36(num):
    chars = "0123456789abcdefghijklmnopqrstuvwxyz"
    if num < 0:
        return "-" + to_base36(-num)
    elif num < 36:
        return chars[num]
    else:
        result = ""
        while num:
            num, rem = divmod(num, 36)
            result = chars[rem] + result
        return result

# aa 对应的对象，key 为各字段名，value 为 index 转换成 base36 后的字符串
aa = { key: to_base36(i) for i, key in enumerate([
    "courseId", "activityId", "activityType", "data", "rollcallId",
    "groupSetId", "accessCode", "action", "enableGroupRollcall", "createUser",
    "joinCourse"
]) }

# ua 对应的对象，key 为各字段名，value 为 na 加上 (index+2) 转为 base36 的字符串
ua = { key: na + to_base36(i + 2) for i, key in enumerate([
    "classroom-exam", "feedback", "vote"
]) }

# ca 为 aa 的键值对反转后的字典
ca = { v: k for k, v in aa.items() }
# sa 为 ua 的键值对反转后的字典
sa = { v: k for k, v in ua.items() }

def parse_sign_qr_code(t):
    """
    解析字符串 t，将其按照分隔符处理后，返回一个字典对象。

    分隔逻辑：
      - 首先以 "!" 分割字符串，再过滤掉空项
      - 对每一段，再以 "~" 分割为 key 和 value 两部分
      - key 使用 ca 映射（若存在对应关系），否则原样使用
      - value 的处理：
          * 如果以 na 开头：
              - 若等于 ia，则返回 True
              - 否则若不等于 oa，则返回 sa 映射中的值（若存在），否则原样返回
              - 若等于 oa，则返回 False
          * 如果以 ra 开头：
              - 截取 ra 之后的部分，用 "." 分割，然后将每部分以 base36 转换为整数
              - 若转换后的列表长度大于1，则取前两项拼接成浮点数返回，否则返回该整数
          * 其它情况：
              - 替换 value 中所有 ea 为 "~"，所有 ta 为 "!"
    """
    result = {}
    if t and isinstance(t, str):
        # 使用 "!" 分割并过滤空字符串
        for part in filter(None, t.split("!")):
            splitted = part.split("~", 1)  # 仅分割成两部分
            if len(splitted) >= 2:
                r, i = splitted[0], splitted[1]
                key = ca.get(r, r)
                # 处理 value
                if i.startswith(na):
                    if i == ia:
                        value = True
                    elif i != oa:
                        value = sa.get(i, i)
                    else:
                        value = False
                elif i.startswith(ra):
                    parts_ = i[1:].split(".")
                    try:
                        nums = [int(part, 36) for part in parts_]
                    except Exception:
                        nums = []
                    if len(nums) > 1:
                        value = float(f"{nums[0]}.{nums[1]}")
                    elif nums:
                        value = nums[0]
                    else:
                        value = i
                else:
                    value = i.replace(ea, "~").replace(ta, "!")
                result[key] = value
    return result

def scan_url_analysis(e: str):
    print(f"scanUrlAnalysis url: {e}")

    # 如果 URL 包含 "/j?p=" 且不是以 "http" 开头，拼接基础 URL
    if "/j?p=" in e and not e.startswith("http"):
        e = base_url + e

    # 如果仍然不是 HTTP 链接，直接返回
    if not e.startswith("http"):
        return e

    try:
        n = urlparse(e)
    except Exception:
        return e

    # 处理特定路径
    if n.path in ["/j", "/scanner-jumper"]:
        o = parse_qs(n.query)
        r = None
        try:
            a = o.get("_p", [None])[0]
            if a:
                r = json.loads(a)
        except Exception:
            pass

        if not r:
            p_value = o.get("p", [""])[0]
            r = parse_sign_qr_code(p_value)

        return json.dumps(r) if r and isinstance(r, dict) and r else e

    return e
```
