### Angular 基本介紹.
AngularJS 是一個 Javascript 的前端框架，藉著自訂 HTML Tag 或 Attribute 來擴展 HTML 的功能並建立成自己的 Component 來達到模組化。

另外最重要的特色為 Template 與 Model 是自動的 two-way binding，也就是異動 Model 時會自動更新 Template 上相對的欄位而反之亦然，這樣的特色得以讓工程師從 JQuery style 必須處理所有繁瑣細節中解脫，變成一次只需要專注於資料的操作或畫面的設計即可。

以下將開始依序介紹 AngularJS 的幾個重要功能，看完本教學後各位應該就能夠將 AngularJS 應用在各種專案中。

### Template 與 Data Binding

    <div ng-app ng-init="qty=1;cost=2">
        <b>Invoice:</b>

        <div>
            Quantity: <input type="number" min="0" ng-model="qty">
        </div>

        <div>
            Costs: <input type="number" min="0" ng-model="cost">
        </div>

        <div>
            <b>Total:</b> {{qty * cost | currency}}
        </div>
    </div>

這是一個基本的 Model 與 Template binding 的模式，Model 在 AngularJS 被稱為 Scope，往後的說明我們都會改用 Scope 代替 Model。
裡面所看到的所有 ng-xxx 都是 AngularJS 內建的元件，又被稱為 Directive，Directive 的詳細介紹我們留到稍後再做說明，這裡僅介紹有用到的 Directive。

第一個看到的是ng-app，用來標注這個 Tag 以及其子節點需要 AngularJS 幫我們控管與處理，頁面上最少會有一個ng-app。

ng-init 內可以執行一些基本的 script，原則上不太會使用到這個 Directive，這裡是用來初始化 model 的值。

再往下看會看到兩個 input 上都有 ng-model，這是重要的 Directive，任何 type 的 input 欄位加上 ng-model 後，input 的值就會和 ng-model 裡定義的 Scope 欄位名稱自動作 binding。

最後看到 {{ }}，這是個 Expressions，Expressions 內的語法跟 Javascript 差不多, 可以執行運算或呼叫 Function，和 Javascript 的差別為以下幾點：

- Context: Expressions 的 context 是 Scope，Javascript 是 window
- Null or Undefined Value 錯誤: Expressions 內如果有欄位是 null 或 undefined 不會造成錯誤，Javascript則會丟出 Error
- Flow Control: Expression 內無法使用任何條件判斷式、迴圈以及 try catch
- Expressions 內無法宣告 Function，Regexp，也無法使用逗點符號以及 void
- Expressions 可以加上 Filters 幫 結果值做額外的處理

    {{qty * cost | currency}}

這裡的 Expressions 就是把 model 內的 qty 和 cost 相乘後再交給 currency 這個 Filter 加上 $ 字號。

內建的 [FIlter](https://docs.angularjs.org/api/ng/filter) 和自己建立 [Custom Filter](https://docs.angularjs.org/guide/filter) 都相當簡單，請自行查詢，本教學不多做解釋。

這樣就完成了一個基本的計算功能，不需要操作 DOM 做取值與賦予值的動作，下圖是此範例的 binding 邏輯。

<img src="https://docs.angularjs.org/img/guide/concepts-databinding1.png"/>

### Controller

    <script type="text/javascript">
        var app = angular.module("myApp", [])
        app.controller("myController", ['$scope', function($scope){
            $scope.qty = 1;
            $scope.cost = 2;
        }])
    </script>

    <div ng-app="myApp" ng-controller="myController">
        <b>Invoice:</b>

        <div>
            Quantity: <input type="number" min="0" ng-model="qty">
        </div>

        <div>
            Costs: <input type="number" min="0" ng-model="cost">
        </div>

        <div>
            <b>Total:</b> {{qty * cost | currency}}
        </div>
    </div>

雖然不一定要有 module 才能使用 controller，但為了方便往後程式擴展的維護，還是習慣都使用 module 比較好

所以首先我們給 ng-app 一個 myApp 的值，表示內容交由 myApp 這個 Module 處理

Module 的宣告方法如下：

    angular.module( "moduleName", [dependencies, ...])

請記得就算不需要 reqiure 其他 module, 第二個參數也需要給空陣列[]，如果沒有給第二個參數會變成get module，而不是 define module。

然後加上ng-controller，加上 ng-controller 後 Angular 會幫我們在這個 node 上建立一個 scope，稍後我們可以在 Controller 內將這個 scope inject 進來並賦予初始值。

Controller 的宣告方法如下：

    app.controller( "controllerName", function(serviceName, ...){
        ...
    })

但上述方法如果有使用 ugilfy 或 closure compiler 的話 serviceName 會編譯成亂碼而找不到正確要 inject 的 service 名稱變得無法執行，所以建議用下述比較嚴謹的方法

    app.controller( "controllerName", ['serviceName', ..., function(serviceNameObj, ...){
        ...
    }])

Angular 內建的 servie 相當多，可以自行到 [API](https://docs.angularjs.org/api) 查詢，我們需要 inject 是 $scope 這個 service，所以把他 inject 進來

    app.controller("myController", ['$scope', function($scope){
        ...
    }])

Controller 可以視為 constructor， 所以初始值的賦予都會寫在這裡，我們開始設定 scope 的初始值

    $scope.qty = 1;
    $scope.cost = 2;

因為 template 和 scope 是自動的 two-way binding，template 上應該一開始就會有我們在這裡設定的值了，這樣就完成了加上 Controller的動作。

下面是加上 Controller 後的邏輯示意圖

<img src="https://docs.angularjs.org/img/guide/concepts-databinding2.png"/>

注意到這個示意圖的 ng-controller 內是用 controllerName as aliasName 的形式，所以他的 Controller 宣告內沒有使用 $scope 這個 service，而是改用 Controller 本身 this 來代替，binding邏輯是一樣的，想了解這個用法的話可以 google controller as 或者是 bindToController。

接下來我們就可以試著把 cost 的值改成由 AJAX 的方式抓取，angular 的 AJAX service 是 $http，所以 controller 可以改成

     app.controller("myController", ['$scope', '$http', function($scope, $http){
         $scope.qty = 1;
         $scope.cost = $http.get("urlPath");
     }])

$http service 會回傳一個 $q 的物件，[$q](https://docs.angularjs.org/api/ng/service/$q) 是 Angular 內建的 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) service，當 AJAX 的結果回傳後 scope 會自動解析出 Promise 的值並一樣會自動 binding 到 template 上。

### Service

我們已經使用過了 Angular 內建的 $scope 和 $http service 了，現在來了解一下 service 大概的特性與用途。

service 的宣告方式如下：

    app.factory("serviceName", ['requiredServiceName', ... , function(requiredServiceObj, ...){
        ...
    }])

和宣告 Controller 差不多，僅差別於使用的是 factory 這個 API, 而在 function 裡回傳了什麼就決定了這個 service 被 inject 的時候可以得到的東西。

基於這個特性，我們可以用 service 來宣告物件

    app.factory("myClassService", [function(){
        function myClass(){ }
        myClass.prototype.constructor = myClass;

        return myClass;
    }])

在不同的 Controller 間分享資料

    app.factory("myDataService", [function(){
        return {
            data: [{id: 1, value: "one"}, {id: 5, value: "fifth"}]
        }
    }])

或者包裝複雜的邏輯，例如之前的 AJAX 取資料範例可以改成 service

    app.factory("myAjaxService", ['$http', 'q', function($http, $q){
        return {
            get: function(){
                return $q(function(resolve, reject){
                    $http.get("urlPath")
                    .then(function(result){
                        if (result.data.price <= 0){
                            reject("wrong price, should never be 0 or smaller than 0.")
                        }
                        resolve(result.data.price)
                    })
                    .catch(function(err){
                        console.log(err)
                        reject(err)
                    })
                })
            }
        }
    }])

    app.controller("myController", ['$scope', 'myAjaxService', function($scope, myAjaxService){
        $scope.qty = 1;
        $scope.cost = myAjaxService.get();
    }])
