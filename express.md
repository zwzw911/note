1. view cache；控制是否把模板（view）缓存在内存中。开发环境disable（需要编辑view并显示）。生产环境要enable，提升性能。  
  if(app.get('env') === 'dev') {  
	   app.disable('view cache')  
  }  
  if(app.get('env') === 'pro') {  
	   app.enable('view cache')  
  }  
