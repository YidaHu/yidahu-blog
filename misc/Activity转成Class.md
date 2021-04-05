# Activity转成Class

## 前言

当我们在Android开发时,有时候为了简化代码,常用listView来做,需要跳转到某个activity测试相应功能,当我们点击listView的item时,而主界面的listview就封装了每个activity的入口,

我们都知道点击listView的某个item让他弹出一个toast,我们如何点击item让他跳转到一个activity呢?其实也差不多,关键是得把每个activity转成xxxActivity.class,然后通过intent和startActivity()进行跳转即可.

代码如下:

	    private static class ActivityToClass {
				private final Class<? extends android.app.Activity> demoClass;
		
				public DemoInfo(Class<? extends android.app.Activity> demoClass) {
					this.demoClass = demoClass;
				}
			}
		//把每个activity转成class
		private static class DemoInfo {
			private final Class<? extends android.app.Activity> demoClass;
	
			public DemoInfo(Class<? extends android.app.Activity> demoClass) {
				this.demoClass = demoClass;
			}
		}
	
		//把每个activity转成xxx.class
		private static final DemoInfo[] demos = {
				new DemoInfo(TextViewMainActivity.class),
				new DemoInfo(ButtonMainActivity.class),
				
		};

使用代码:

	demos[i].demoClass