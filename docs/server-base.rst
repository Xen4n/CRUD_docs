.. _server-base:

Наполнение сервера
=====================

После настройки сервера нам нужно привязать к нему ранее созданную базу.

Привязка к базе
----------------

В директории **main** -> **resources** создадим файл ``application.properties`` и запишем в него
конфигурацию доступа к нашей базе MySQL:
::

	jdbc.driverClassName = com.mysql.jdbc.Driver
	jdbc.url = jdbc:mysql://localhost:3306/cars_db
	jdbc.username = root
	jdbc.password = 0000

Теперь укажем эти настройки серверу. В пакете ``com.scrud.config`` создадим класс ``DataSourceConfig``:

.. code-block:: java
	:linenos:

	package com.scrud.config;

	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.context.annotation.PropertySource;
	import org.springframework.core.env.Environment;
	import org.springframework.jdbc.datasource.DriverManagerDataSource;
	import javax.sql.DataSource;
	
	@Configuration
	@PropertySource(value = {"classpath:application.properties"})
	public class DataSourceConfig {
		@Autowired
		private Environment environment;
	
		@Bean
		public DataSource getDataSource() {
			DriverManagerDataSource dataSource = new DriverManagerDataSource();
			dataSource.setDriverClassName(environment.getRequiredProperty("jdbc.driverClassName"));
			dataSource.setUrl(environment.getRequiredProperty("jdbc.url"));
			dataSource.setUsername(environment.getRequiredProperty("jdbc.username"));
			dataSource.setPassword(environment.getRequiredProperty("jdbc.password"));
			return dataSource;
		}
	
	}
	
	
Отлично, теперь наш сервер знает где находиться база и как к ней обратиться. 
Теперь давайте наполним его задуманным функционалом, и реализуем выполнение
запросов в базу данных.


Создание сущностей
-------------------

Для начала нам необходимо создать сущности марки и модели автомобилей, этими данными сервер будет
обмениваться с базой. В пакете ``com.scrud`` создадим новый пакет ``model``. 
В него поместим класс ``CarBrand``:

.. code-block:: java
	:linenos:

	package com.scrud.model;

	import org.hibernate.validator.constraints.NotEmpty;
	import javax.persistence.*;
	import java.io.Serializable;
	
	@Entity
	@Table(name = "brand")
	@NamedQuery(name = "CarBrand.findAll", query = "select c from CarBrand c")
	
	public class CarBrand implements Serializable {
		@Id @GeneratedValue(strategy= GenerationType.AUTO)
		private int idBrand;
		@NotEmpty @Column(unique=true, nullable=false)
		private String name;

		@Column private short foundedYear;
	
		@Column private String headquarter;
	
		public CarBrand() {}
	
		public void setIdBrand(int idBrand) {
			this.idBrand = idBrand;
		}
	
		public void setName(String name) {
			this.name = name;
		}
	
		public void setFoundedYear(short foundedYear) {
			this.foundedYear = foundedYear;
		}
	
		public void setHeadquarter(String headquarter) {
			this.headquarter = headquarter;
		}
	
		public int getIdBrand() {
			return idBrand;
		}
	
		public String getName() {
			return name;
		}
	
		public short getFoundedYear() {
			return foundedYear;
		}
	
		public String getHeadquarter() {
			return headquarter;
		}
	
		@Override
		public String toString() {
			return "CarBrand{" +
					"idBrand=" + idBrand +
					", name='" + name + '\'' +
					", foundedYear=" + foundedYear +
					", headquarter='" + headquarter + '\'' +
					'}';
		}
	}
	
И туда же поместим класс ``CarModel``:

.. code-block:: java
	:linenos:
	
	package com.scrud.model;
	
	import org.hibernate.validator.constraints.NotEmpty;
	import javax.persistence.*;
	import java.io.Serializable;
	
	@Entity
	@Table(name="model")
	@NamedQuery(name="CarModel.findAll", query="select m from CarModel m")
	
	public class CarModel implements Serializable {
		@Id @GeneratedValue(strategy= GenerationType.AUTO)
		private int idModel;
	
		@Column(insertable = false, updatable = false)
		private int idBrand;
	
		@ManyToOne
		@JoinColumn(name = "idBrand")
		private CarBrand carBrand;
	
		@NotEmpty @Column
		private String modelName;
		@Column private String generation;
		@Column private short productionYear;
		@Column private byte doors;
		@Column private byte seats;
		@Column private short maximumSpeed;
	
		public CarModel() {}
	
		public int getIdModel() {
			return idModel;
		}
	
		public int getIdBrand() {
			return idBrand;
		}
	
		public CarBrand getCarBrand() {
			return carBrand;
		}
	
		public String getModelName() {
			return modelName;
		}
	
		public String getGeneration() {
			return generation;
		}
	
		public short getProductionYear() {
			return productionYear;
		}
	
		public byte getDoors() {
			return doors;
		}
	
		public byte getSeats() {
			return seats;
		}
	
		public short getMaximumSpeed() {
			return maximumSpeed;
		}
	
		public void setIdModel(int idModel) {
			this.idModel = idModel;
		}
	
		public void setIdBrand(int idBrand) {
			this.idBrand = idBrand;
		}
	
		public void setCarBrand(CarBrand carBrand) {
			this.carBrand = carBrand;
		}
	
		public void setModelName(String modelName) {
			this.modelName = modelName;
		}
	
		public void setGeneration(String generation) {
			this.generation = generation;
		}
	
		public void setProductionYear(short productionYear) {
			this.productionYear = productionYear;
		}
	
		public void setDoors(byte doors) {
			this.doors = doors;
		}
	
		public void setSeats(byte seats) {
			this.seats = seats;
		}
	
		public void setMaximumSpeed(short maximumSpeed) {
			this.maximumSpeed = maximumSpeed;
		}
	
		@Override
		public String toString() {
			return "CarModel{" +
					"idModel=" + idModel +
					", carBrand=" + carBrand.getName() +
					", idBrand=" + idBrand +
					", modelName='" + modelName + '\'' +
					", generation='" + generation + '\'' +
					", productionYear=" + productionYear +
					", doors=" + doors +
					", seats=" + seats +
					", maximumSpeed=" + maximumSpeed +
					'}';
		}
	}
	
В этих классах с помощью аннотаций уже указаны необходимые запросы в базу и соответствия между полями.

`Скачаь ресурсы </_static/resources.7z>`_
