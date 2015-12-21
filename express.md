1. view cache；控制是否把模板（view）缓存在内存中。开发环境disable（需要编辑view并显示）。生产环境要enable，提升性能。  
  if(app.get('env') === 'dev') {  
	   app.disable('view cache')  
  }  
  if(app.get('env') === 'pro') {  
	   app.enable('view cache')  
  }  
  
2. view engine  
在render的时候，如果没有后缀名，则默认使用view engine指定的模板。
app.set('view engine','jade')
res.render('index')==>使用jade
res.render('index.ejs')===>使用ejs


