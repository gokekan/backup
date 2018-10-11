# 정규식을 이용한 jQuery 셀렉터

```javascript
$('div').filter(function() { 
    return this.id.match(/tab_[0-9]/); 
}).hide();
```

div 태그 중에서 id가 "tab_"이고 뒷자리에 숫자가 붙은 셀렉터를 숨김처리한다.
