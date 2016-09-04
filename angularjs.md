1. js中包含一个以上的app（html中有一个以上ng-app），除了第一个app是angular自动启动，剩下的需要手工boot（angular.bootstrap(document.getElementById("bgCover"),['cropApp']);）
2. 不同的contrller中，factory不能共享状态（即，不同的controller中，factory是独立的）
3. 使用event是，需要在ejs中把$event作为参数传递，并且在controller中把$event作为参数。  
\<a ng-click="click(**$event**)"\>\</a\>  
$scope.click=function(**$event**){}  

##### ui-router  
1. **去除url中的#**  
  1.1 在config中enable HTML5 mode。`$locationProvider.html5Mode(true);`  
  1.2 在html的head中添加`<base href="/" />`  
2. 将template所在目录设为express的**静态路径**(获取template就像获取css/js/image一样)。  
3. ui-sref: 点击某个元素，对应触发的状态。如果对应是子模板（state为parent.child），那么ui-sref也要写成parent.child格式，而不是仅仅child，让angular根据html结果来猜测parent。    


