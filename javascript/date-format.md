# 日期格式化处理

平时用到的日期处理方法

```javascript
  format: function(date, format = 'yyyy-MM-dd') {
    var o = {
      'M+': date.getMonth() + 1, //month
      'd+': date.getDate(), //day
      'h+': date.getHours(), //hour
      'm+': date.getMinutes(), //minute
      's+': date.getSeconds(), //second
      'q+': Math.floor((date.getMonth() + 3) / 3), //quarter
      S: date.getMilliseconds(), //millisecond
      w: '日一二三四五六'.charAt(date.getDay())
    }

    format = format.replace(/y{4}/, date.getFullYear()).replace(
      /y{2}/,
      date
        .getFullYear()
        .toString()
        .substring(2)
    )

    var k, reg
    for (k in o) {
      reg = new RegExp(k)
      format = format.replace(reg, match)
    }

    function match(m) {
      return m.length === 1 ? o[k] : ('00' + o[k]).substr(('' + o[k]).length)
    }

    return format
  }
```