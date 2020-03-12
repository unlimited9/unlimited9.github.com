
```  
// SPA 모듈 작성 순서 예시  
const _module = (function() {  
  
    "use strict";  
  
 const root = this;  
  
  // 1. 모듈 스코프 내에서 사용할 변수 작성  
  let scopeVar = {};  
 let utilMethod;  
 let manipulateDom;  
 let eventHandle;  
 let initModule;  
  
  // 2. 유틸리티 메소드 작성  
  utilMethod = function() {  
        // 실행코드  
  };  
  
  // 3. DOM 조작 메소드 작성  
  manipulateDom = function() {  
        // 실행코드  
  };  
  
  // 4. 이벤트 핸들러 작성  
  eventHandle = function() {  
        // 실행코드  
  };  
  
  // Public 메소드 작성  
  initModule = function() {  
        // 실행코드  
  };  
  
 return {  
        init : initModule  
    };  
})();  
  
  
// call 함수 이용  
(function () {  
    'use strict';  
 const root = this;  
 const version = '1.0';  
  
 let _module_01;  
 if (typeof exports !== 'undefined') {  
        _module_01 = exports;  
  } else {  
        _module_01 = root._module_01 = {};  
  }  
    _module_01.getVersion = function () {  
        return version;  
  }  
}).call(this); //}).apply(this);  
console.log(_module_01.getVersion());  
  
// 글로벌 객체를 파라미터로 전달  
(function (global) {  
    const root = global;  
 const version = '1.0';  
  
 let _module_02;  
 if (typeof exports !== 'undefined') {  
        _module_02 = exports;  
  } else {  
        _module_02 = root._module_02 = {};  
  }  
    _module_02.getVersion = function () {  
        return version;  
  }  
}(this));  
console.log(_module_02.getVersion());  
  
// 즉시실행함수 내부에서 글로벌 객체를 내부 변수에 할당  
(function () {  
    const root = this;  
 const version = '1.0';  
  
 let _module_03;  
 if (typeof exports !== 'undefined') {  
        _module_03 = exports;  
  } else {  
        _module_03 = root.Module3 = {};  
  }  
    _module_03.getVersion = function () {  
        return version;  
  }  
}());  
console.log(_module_03.getVersion());  
  
  
// 기명 즉시실행함수 이용  
let _module_04 = (function () {  
    const root = this;  
 const version = '1.0';  
  
 let _module_04;  
 if (typeof exports !== 'undefined') {  
        _module_04 = exports;  
  } else {  
        _module_04 = root.Module = {};  
  }  
    _module_04.getVersion = function () {  
        return version;  
  }  
    return _module_04;  
}());  
console.log(_module_04.getVersion());
```
