1. js中包含一个以上的app（html中有一个以上ng-app），除了第一个app是angular自动启动，剩下的需要手工boot（angular.bootstrap(document.getElementById("bgCover"),['cropApp']);）
2. 不同的contrller中，factory不能共享状态（即，不同的controller中，factory是独立的）
