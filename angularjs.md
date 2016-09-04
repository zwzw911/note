1. js中包含一个以上的app（html中有一个以上ng-app），除了第一个app是angular自动启动，剩下的需要手工boot（angular.bootstrap(document.getElementById("bgCover"),['cropApp']);）
2. 不同的contrller中，factory不能共享状态（即，不同的controller中，factory是独立的）
3. 使用event是，需要在ejs中把$event作为参数传递，并且在controller中把$event作为参数。  
\<a ng-click="click(**$event**)"\>\</a\>  
$scope.click=function(**$event**){}  

##### 去除url中的#（使用ui-router或者$router）  
1. 在config中enable HTML5 mode。`$locationProvider.html5Mode(true);`  
2. 在html的head中添加`<base href="/" />`  
