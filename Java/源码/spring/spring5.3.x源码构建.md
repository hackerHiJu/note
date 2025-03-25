## spring 5.3.x源码构建

### 一、fork spring仓库

把spring的仓库导入到自己的gitee仓库中，拉去速度更快。下面是我自己的仓库地址

拉取出5.3.x版本的spring源码

**git clone -b 5.3.x https://gitee.com/haijun1998/spring-framework.git**

### 二、安装gralde

根据编译spring版本来决定使用什么版本的gradle。查看源码下 **\gradle\wrapper\gradle-wrapper.properties** 中需要什么版本

![1641477074726](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477074726.png)

下载好gradle-7.2版本，配置环境变量

![1641477115375](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477115375.png)

防止spring每次编译都去下载gradle安装包，将distributionUrl改成本地文件路径

![1641477074726](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477037289.png)

### 三、编译spring

导入spring源码到idea中，加入依赖

```gradle
buildscript { 
	repositories { 
		maven { url "https://repo.spring.io/plugins-release" } 
	} 
 }
```

![1641477375151](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477503881.png)

![1641477385928](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477610683.png)

```gradle
repositories {
	//新增以下2个阿里云镜像 
	maven { url 'https://maven.aliyun.com/nexus/content/groups/public/' } 
	maven { url 'https://maven.aliyun.com/nexus/content/repositories/jcenter' } 		     mavenCentral() 
	maven { url "https://repo.spring.io/libs-spring-framework-build" } 
	maven { url "https://repo.spring.io/milestone" }  
	//新增spring插件库 
	maven { url "https://repo.spring.io/plugins-release" } 
}
```

设置项目sdk版本

![1641477503881](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477626171.png)

**找到spring-oxm以及spring-core下的compileTestjava进行编译**

编译成功后，编译整个工程，spring>build>build

![1641477564279](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477375151.png)

### 四、创建新的子模块

![1641477610683](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477385928.png)

引入依赖

![1641477626171](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641478066239.png)

**再次编译spring项目，成功后运行测试代码**

```java
@Configuration
public class ApplicationTestDemo {

	public static void main(String[] args) {
		System.out.println("hello");
		AnnotationConfigApplicationContext configApplicationContext = new AnnotationConfigApplicationContext(ApplicationTestDemo.class);
		A a = configApplicationContext.getBean(A.class);
		System.out.println(a.name);
	}

	@Bean
	public A a(){
		return new A();
	}

	public static class A {
		public String name = "我是张三啊";
	}

}
```

**最后可以去掉spring模块下面的 src/checkstyle/checkstyle.xml**，内容注释掉，风格检查

![1641477751090](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477564279.png)

### 五、异常问题

![1641478038222](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641477751090.png)

![1641478066239](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641478038222.png)

### 六、模块说明

![1641478101533](https://cdn.jsdelivr.net/gh/a1424132610/note-picture@main/note-picture/1641478101533.png)

