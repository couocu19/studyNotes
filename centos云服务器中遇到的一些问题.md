# centos云服务器中遇到的一些问题

## 4.27更新

- ### 关于端口被占用的问题

  所报的错误：

  ```
  # Creating Server TCP listening socket *:6379: bind: Address already in use--
  ```

  解决方案：

  ```
  1.找到占用redis的进程
  ps -ef | grep -i redis
  2.查看进程编号
  root      3086     1  0 Apr24 ?        00:00:07 ./bin/redis-server *:6379        
  root      3531  3467  0 01:00 pts/0    00:00:00 grep -i redis  
  3.杀死该进程
  ②杀死该进程;
  使用kill 命令
  输入命令：
  kill -9 3086  
  4.重新启动redis服务器
  ./redis-server 
  ```
  
- ### 关于springboot项目部署在云服务器上出现的问题

  昨晚和今晚努力将springboot项目传到云服务器上，出现了不少问题。

  #### 问题1：

  打包时有一个手动导入的jar包提示找不到，解决方法需要在pom文件中加一些额外配置

  ```xml
  <plugin>
  	<groupId>org.apache.maven.plugins</groupId>
  	<artifactId>maven-compiler-plugin</artifactId>
  	<configuration>
  	<source>1.8</source>
  	<target>1.8</target>
  	<!--根据你把lib放的位置-->
  	<compilerArguments>
  		<extdirs>${project.basedir}/src/main/resources/lib</extdirs>
  	</compilerArguments>
  		</configuration>
  			</plugin>
  			<plugin>
  				<groupId>org.apache.maven.plugins</groupId>
  				<artifactId>maven-jar-plugin</artifactId>
  				<configuration>
  					<archive>
  						<manifest>
  							<addClasspath>true</addClasspath>
  							<useUniqueVersions>false</useUniqueVersions>
  							<classpathPrefix>lib/</classpathPrefix>
  							<mainClass>cn.mymaven.test.TestMain</mainClass>
  						</manifest>
  					</archive>
  				</configuration>
  			</plugin>
  			<plugin>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-maven-plugin</artifactId>
  			<configuration>
  				<mainClass>com.poets.PoetProjectApplication</mainClass>
  			</configuration>
  </plugin>
  ```

  #### 问题2：

  可以打包成功但是在云服务器上跑的时候报错提示项目找不到主类，所以pom文件总关于maven的配置需要加上主类的配置

  ```xml
  	<plugin>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-maven-plugin</artifactId>
  			<configuration>
  				<mainClass>com.poets.PoetProjectApplication</mainClass>
  			</configuration>
      </plugin>
  ```

  #### 问题3：

  可以运行但是！一旦关闭服务器之后项目就停止运行了，这时在启动项目时需要这样来启动

   **这里的&不能省略 ，表示守护进程的意思，运行在后台。** 

  ```
  [root@izbp15p4bkvr39g2jyaa9qz webapps]# nohup java -jar poets-0.0.1-SNAPSHOT            .jar &
  [2] 23081
  [root@izbp15p4bkvr39g2jyaa9qz webapps]# nohup: ignoring input and appending          output to ‘nohup.out’
  ```

  

