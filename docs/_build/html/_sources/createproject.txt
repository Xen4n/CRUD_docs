.. _createproject:

Создание проекта
==================

Для начала нам понадобится установленная `Java (JDK) <http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html>`_,
а также среда разработки. Мы советуем использовать `IntelliJ IDEA <https://www.jetbrains.com/idea/download/#section=windows>`_.

Запускаем IntelliJ IDEA и создаем новый проект. В окне выбора типа проекта указываем Maven.

.. image:: _static/proj_1.PNG
       :height: 657px
       :width: 738 px
       :scale: 100%
       :alt: Maven
       :align: center
	   
На следующей странице в поле **GroupId** вводим название фирмы (группы), а также артефакт проекта в поле **ArtifactId**.

Далее вводим название проекта и место расположения на диске. После этого откроется окно проекта. Структура проекта будет выглядеть примерно так: 

.. image:: _static/proj_2.PNG
       :height: 300px
       :width: 479 px
       :scale: 100%
       :alt: Project
       :align: center
	   
На следующем шаге нам необходимо отредактировать ``pom.xml`` файл, указав правильные зависимости.
Нижеуказанный код поместим между тегами ``</version>`` и ``</project>``:

.. code-block:: xml
	:linenos:
	
	<properties>
		<spring.version>4.2.1.RELEASE</spring.version>
		<hibernate.version>4.2.5.Final</hibernate.version>
		<mysql.connector.version>5.1.31</mysql.connector.version>
		<jpa.version>1.0.0.Final</jpa.version>
	</properties>
	
	<packaging>war</packaging>
	
	<dependencies>
		<!--SPRING-->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>
	
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>
	
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
	
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>4.2.1.RELEASE</version>
		</dependency>
	
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${spring.version}</version>
		</dependency>
	
		<!-- Hibernate -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>${hibernate.version}</version>
		</dependency>
	
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${hibernate.version}</version>
		</dependency>
	
		<dependency>
			<groupId>org.hibernate.javax.persistence</groupId>
			<artifactId>hibernate-jpa-2.1-api</artifactId>
			<version>${jpa.version}</version>
		</dependency>
		<!-- jsr303 validation -->
		<dependency>
			<groupId>javax.validation</groupId>
			<artifactId>validation-api</artifactId>
			<version>1.1.0.Final</version>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-validator</artifactId>
			<version>5.1.3.Final</version>
		</dependency>
	
		<!-- MySQL -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>${mysql.connector.version}</version>
		</dependency>
	
		<!-- Servlet+JSP+JSTL -->
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>javax.servlet.jsp-api</artifactId>
			<version>2.3.1</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
	
		<!-- Pdf library -->
		<dependency>
			<groupId>com.itextpdf</groupId>
			<artifactId>itextpdf</artifactId>
			<version>5.0.6</version>
		</dependency>
		<!--Apache POI library-->
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi</artifactId>
			<version>3.13</version>
		</dependency>
	
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml</artifactId>
			<version>3.13</version>
		</dependency>
	
		<!--WebJars-->
		<dependency>
			<groupId>org.webjars</groupId>
			<artifactId>bootstrap</artifactId>
			<version>3.1.0</version>
			<exclusions>
				<exclusion>
					<groupId>org.webjars</groupId>
					<artifactId>jquery</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	
		<dependency>
			<groupId>org.webjars</groupId>
			<artifactId>jquery</artifactId>
			<version>1.9.0</version>
		</dependency>
	
	</dependencies>
	
	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-war-plugin</artifactId>
					<version>2.4</version>
					<configuration>
						<warSourceDirectory>src/main/webapp</warSourceDirectory>
						<warName>scrudop</warName>
						<failOnMissingWebXml>false</failOnMissingWebXml>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
		<finalName>SCRUDoperations</finalName>
	</build>
	
Теперь добавим поддержку Spring: :menuselection:`Правой клавишей мыши по корню проекта --> ...Add Framework Support`.
В Появившемся окне выбираем :menuselection:`Spring --> Spring MVC`. Должна быть выбрана метка **Use library**, жмем ОК.

Теперь в папке Java создадим пакет ``com.scrud.configs``. В нём создадим новый класс ``WebConfig`` и заполним его следующим кодом:

.. code-block:: java
	:linenos:
	
	package com.scrud.config;

	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.web.servlet.ViewResolver;
	import org.springframework.web.servlet.config.annotation.EnableWebMvc;
	import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
	import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
	import org.springframework.web.servlet.view.InternalResourceViewResolver;
	import org.springframework.web.servlet.view.ResourceBundleViewResolver;
	
	@Configuration
	@ComponentScan({"com.scrud"})
	@EnableWebMvc
	public class WebConfig extends WebMvcConfigurerAdapter {
	
		@Bean
		public ViewResolver getViewResolver(){
			InternalResourceViewResolver resolver = new InternalResourceViewResolver();
			resolver.setPrefix("/pages/");
			resolver.setSuffix(".jsp");
			resolver.setOrder(2);
			return resolver;
		}
	
		@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
			if (!registry.hasMappingForPattern("/webjars/**")) {
				registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META-INF/resources/webjars/");
			}
		}
	
	}

В том же пакете добавим ещё один класс ``AppInit`` с таким содержанием:

.. code-block:: java
	:linenos:
	
	package com.scrud.config;

	import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;
	
	public class AppInit extends AbstractAnnotationConfigDispatcherServletInitializer {
		// Этот метод должен содержать конфигурации которые инициализируют Beans
		// для инициализации бинов у нас использовалась аннотация @Bean
		@Override
		protected Class<?>[] getRootConfigClasses() {
			return new Class<?>[]{
					WebConfig.class
			};
		}
	
		// Тут добавляем конфигурацию, в которой инициализируем ViewResolver
		@Override
		protected Class<?>[] getServletConfigClasses() {
	
			return new Class<?>[]{
					WebConfig.class
			};
		}
	
		@Override
		protected String[] getServletMappings() {
			return new String[]{"/"};
		}
	
	}
	
Теперь нам нужно применить новые конфигурации к проекту. Для этого идем в пункт меню 
:menuselection:`File --> Project Structure...`, выбираем в списке :menuselection:`Project Settings
--> Modules`  и раскрываем содержание модуля CRUD

.. image:: _static/proj_3.PNG
       :height: 329px
       :width: 336px
       :scale: 100%
       :alt: Project structure
       :align: center

Далее выделяем **Spring** и нажимаем *зеленый плюс* (добавить). В окне **New Application Context**
ставим галочку напротив созданного нами класса ``WebConfig.java``, жмем ОК.

.. image:: _static/proj_4.PNG
       :height: 281px
       :width: 429px
       :scale: 100%
       :alt: Modules 1
       :align: center

После чего выделяем **Web**. В нижнем разделе **Web Resource Directories** делаем двойной клик по строке,
заканчивающейся на \webapp и в открывшемся окне нажимаем ОК, соглашаясь с созданием новой директории.

.. image:: _static/proj_5.PNG
       :height: 363px
       :width: 702px
       :scale: 100%
       :alt: Modules 1
       :align: center
	   
Теперь просто нажимаем **Apply**  и **OK**. 
Мы сконфигурировали наш сервер. Осталось добавить контроллер, который будет выдавать ответ на запрос 
и сам ответ, то есть страницу, которую увидет пользователь. 

Создадим новый пакет ``com.scrud.controllers`` в котором создадим класс ``HelloController``:

.. code-block:: java
	:linenos:
	
	package com.scrud.controllers;

		import org.springframework.stereotype.Controller;
		import org.springframework.ui.ModelMap;
		import org.springframework.web.bind.annotation.RequestMapping;
		import org.springframework.web.bind.annotation.RequestMethod;
		
		@Controller
		public class HelloController {
		
		
			@RequestMapping(value = "/", method = RequestMethod.GET)
			public String printWelcome(ModelMap model) {
				return "index";
		
			}
		
			@RequestMapping(value = "/about", method = RequestMethod.GET)
			public String printJpa(ModelMap model) {
		
				return "about";
		
			}
		
	}
	
После этого в папке Java создадим пакет ``com.scrud.config``, в нём создадим класс ``AppInit``:

.. code-block:: java
	:linenos:

	package com.scrud.config;
	
	import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;
	
	public class AppInit extends AbstractAnnotationConfigDispatcherServletInitializer {
		// Этот метод должен содержать конфигурации которые инициализируют Beans
		// для инициализации бинов у нас использовалась аннотация @Bean
		@Override
		protected Class<?>[] getRootConfigClasses() {
			return new Class<?>[]{
					WebConfig.class
			};
		}
	
		// Тут добавляем конфигурацию, в которой инициализируем ViewResolver
		@Override
		protected Class<?>[] getServletConfigClasses() {
	
			return new Class<?>[]{
					WebConfig.class
			};
		}
	
		@Override
		protected String[] getServletMappings() {
			return new String[]{"/"};
		}
	
	}