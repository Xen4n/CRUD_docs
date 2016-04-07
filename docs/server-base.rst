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
Теперь необходимо задать серверу логику обработки данных трёма вариантами:

* Hibernate
* JDBC
* JPA

Настройка конфигураций запросов
--------------------------------

В пакет ``com.scrud.config`` добавим 3 класса:

1. ``HibernateConfig``:

.. code-block:: java
	:linenos:
	
	package com.scrud.config;
	
	import com.scrud.dao.CarBrandDAO;
	import com.scrud.dao.CarModelDAO;
	import com.scrud.dao.hibernate.CarBrandDAOHibernateImpl;
	import com.scrud.dao.hibernate.CarModelDAOHibernateImpl;
	import org.hibernate.SessionFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.context.annotation.PropertySource;
	import org.springframework.orm.hibernate4.HibernateTransactionManager;
	import org.springframework.orm.hibernate4.LocalSessionFactoryBean;
	import org.springframework.transaction.annotation.EnableTransactionManagement;
	import javax.sql.DataSource;
	
	@Configuration
	@EnableTransactionManagement
	@PropertySource(value = {"classpath:application.properties"})
	@ComponentScan({"com.scrud"})
	public class HibernateConfig {
	
		@Autowired
		private DataSource dataSource;
	
		//Hibernate configuration
		@Bean
		public LocalSessionFactoryBean sessionFactory() {
			LocalSessionFactoryBean sessionFactory = new LocalSessionFactoryBean();
			sessionFactory.setDataSource(dataSource);
			sessionFactory.setPackagesToScan(new String[] { "com.scrud.model" });
			return sessionFactory;
		}
	
		@Bean
		@Autowired
		public HibernateTransactionManager hibernateTransactionManager(SessionFactory s) {
			HibernateTransactionManager txManager = new HibernateTransactionManager();
			txManager.setSessionFactory(s);
			return txManager;
		}
	
		@Bean
		public CarBrandDAO getCarBrandHibernateDAO() {
			return new CarBrandDAOHibernateImpl();
		}
		@Bean
		public CarModelDAO getCarModelHibernateDAO() {
			return new CarModelDAOHibernateImpl();
		}
	}

2. ``JdbcConfig``:

.. code-block:: java
	:linenos:
	
	package com.scrud.config;
	
	import com.scrud.dao.CarBrandDAO;
	import com.scrud.dao.CarModelDAO;
	import com.scrud.dao.jdbc.CarBrandDAOJdbcImpl;
	import com.scrud.dao.jdbc.CarModelDAOJdbcImpl;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	
	import javax.sql.DataSource;
	
	@Configuration
	public class JdbcConfig {
		@Autowired
		private DataSource dataSource;
	
		@Bean
		public CarBrandDAO getCarBrandJdbcDAO() {
			return new CarBrandDAOJdbcImpl(dataSource);
		}
		@Bean
		public CarModelDAO getCarModelJdbcDAO() {
			return new CarModelDAOJdbcImpl(dataSource);
		}
	}
	
3. ``JpaConfig``:

.. code-block:: java
	:linenos:
	
	package com.scrud.config;

	import javax.persistence.EntityManagerFactory;
	import javax.sql.DataSource;

	import com.scrud.dao.CarBrandDAO;
	import com.scrud.dao.CarModelDAO;
	import com.scrud.dao.jpa.CarBrandDAOJpaImpl;
	import com.scrud.dao.jpa.CarModelDAOJpaImpl;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.context.annotation.PropertySource;
	import org.springframework.orm.jpa.JpaTransactionManager;
	import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
	import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
	import org.springframework.transaction.PlatformTransactionManager;
	import org.springframework.transaction.annotation.EnableTransactionManagement;
	
	@Configuration
	@PropertySource(value = {"classpath:application.properties"})
	@ComponentScan
	@EnableTransactionManagement
	public class JpaConfig {
		@Autowired
		private DataSource dataSource;
	
		@Bean
		public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
			LocalContainerEntityManagerFactoryBean entityManagerFactory =  new LocalContainerEntityManagerFactoryBean();
	
			entityManagerFactory.setDataSource(dataSource);
			entityManagerFactory.setPackagesToScan(new String[] {"com.scrud.model"});
			HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
			entityManagerFactory.setJpaVendorAdapter(vendorAdapter);
	
			return entityManagerFactory;
		}
	
		@Bean
		@Autowired
		public PlatformTransactionManager jpaTransactionManager(EntityManagerFactory emf) {
			JpaTransactionManager txManager = new JpaTransactionManager();
			txManager.setEntityManagerFactory(emf);
			return txManager;
		}
	
		@Bean
		public CarBrandDAO getCarBrandJpaDAO() {
			return new CarBrandDAOJpaImpl();
		}
	
		@Bean
		public CarModelDAO getCarModelJpaDAO() {
			return new CarModelDAOJpaImpl();
		}
	}

Настройка доступа к данными
------------------------------

В пакете ``com.scrud`` создадим новый пакет ``dao``, который будет отвечать за доступ к данным.
В этом пакет поместим 2 интерфейса:

1. ``CarBrandDAO``:

.. code-block:: java
	:linenos:
	
	package com.scrud.dao;
	
	import com.scrud.model.CarBrand;
	import java.util.List;
	
	public interface CarBrandDAO{
	
		void saveOrUpdate(CarBrand item);
	
		void delete(int itemId);
	
		CarBrand get(int itemId);
	
		List<CarBrand> list();
		
	}

2. ``CarModelDAO``:

.. code-block:: java
	:linenos:
	
	package com.scrud.dao;
	
	import com.scrud.model.CarModel;
	
	import java.util.List;
	
	public interface CarModelDAO {
	
		void saveOrUpdate(CarModel item);
	
		void delete(int itemId);
	
		CarModel get(int itemId);
	
		List<CarModel> list();
	
	}

Теперь реализуем эти интерфейсы для каждого варианта доступа. Создадим в пакете ``dao`` 3 новых пакета:
``hibernate``, ``jdbc`` и ``jpa`` для удобного разделения классов.

В пакете ``hibernate`` создадим классы для каждой сущности:

1. ``CarBrandDAOHibernateImpl``:

.. code-block:: java
	:linenos:
	
	package com.scrud.dao.hibernate;
	
	import com.scrud.dao.CarBrandDAO;
	import com.scrud.model.CarBrand;
	import org.hibernate.Criteria;
	import org.hibernate.Session;
	import org.hibernate.SessionFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Repository;
	import java.util.List;
	
	@Repository
	public class CarBrandDAOHibernateImpl implements CarBrandDAO{
	
		@Autowired
		private SessionFactory sessionFactory;
	
		private Session getCurrentSession() {
			return sessionFactory.getCurrentSession();
		}
	
		public void saveOrUpdate(CarBrand item) {
			if (item.getIdBrand() > 0) {
				//update
				getCurrentSession().merge(item);
			} else {
				//insert
				getCurrentSession().save(item);
			}
	
		}
	
		public void delete(int itemId) {
			CarBrand carBrand = get(itemId);
			if (carBrand != null)
				getCurrentSession().delete(carBrand);
		}
	
		public CarBrand get(int itemId) {
			CarBrand carBrand = (CarBrand) getCurrentSession().get(CarBrand.class, itemId);
	
			return carBrand;
		}
	
		public List<CarBrand> list() {
			Criteria criteria = getCurrentSession().createCriteria(CarBrand.class);
	
			return criteria.list();
		}
	}
	
2. ``CarModelDAOHibernateImpl``:

.. code-block:: java
	:linenos:
	
	package com.scrud.dao.hibernate;
	
	import com.scrud.dao.CarModelDAO;
	import com.scrud.model.CarModel;
	import org.hibernate.Criteria;
	import org.hibernate.Session;
	import org.hibernate.SessionFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Repository;
	import java.util.List;
	
	@Repository
	public class CarModelDAOHibernateImpl implements CarModelDAO{
	
		@Autowired
		private SessionFactory sessionFactory;
	
		private Session getCurrentSession() {
			return sessionFactory.getCurrentSession();
		}
	
		public void saveOrUpdate(CarModel item) {
			if (item.getIdModel() > 0) {
				getCurrentSession().merge(item);
			} else {
				getCurrentSession().save(item);
			}
	
		}
	
		public void delete(int itemId) {
			CarModel carModel = get(itemId);
			if (carModel != null)
				getCurrentSession().delete(carModel);
		}
	
		public CarModel get(int itemId) {
			CarModel carModel = (CarModel) getCurrentSession().get(CarModel.class, itemId);
	
			return carModel;
		}
	
		public List<CarModel> list() {
			Criteria criteria = getCurrentSession().createCriteria(CarModel.class);
	
			return criteria.list();
		}
	}

Аналогично в пакете ``jdbc`` создадим 2 соответствующих класса:

1. ``CarBrandDAOJdbcImpl``:

.. code-block:: java
	:linenos:
	
	package com.scrud.dao.jdbc;
	
	import com.scrud.dao.CarBrandDAO;
	import com.scrud.model.CarBrand;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.util.List;
	import javax.sql.DataSource;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.dao.DataAccessException;
	import org.springframework.jdbc.core.JdbcTemplate;
	import org.springframework.jdbc.core.ResultSetExtractor;
	import org.springframework.jdbc.core.RowMapper;
	
	
	public class CarBrandDAOJdbcImpl implements CarBrandDAO {
	
		private JdbcTemplate jdbcTemplate;
		@Autowired
		public CarBrandDAOJdbcImpl(DataSource dataSource) {
			jdbcTemplate = new JdbcTemplate(dataSource);
		}
	
		public void saveOrUpdate(CarBrand item) {
			if (item.getIdBrand() > 0) {
				// update
				System.out.println("CarBrand update");
				String sql = "UPDATE brand SET name=?, foundedYear=?, headquarter=? WHERE idBrand=?";
				jdbcTemplate.update(sql, item.getName(), item.getFoundedYear(),item.getHeadquarter(), item.getIdBrand());
			} else {
				// insert
				System.out.println("CarBrand insert");
				String sql = "INSERT INTO brand (name, foundedYear, headquarter)"
						+ " VALUES (?, ?, ?)";
				jdbcTemplate.update(sql, item.getName(), item.getFoundedYear(), item.getHeadquarter());
			}
		}
	
		public void delete(int itemId) {
			String sql = "DELETE FROM brand WHERE idBrand=?";
			jdbcTemplate.update(sql, itemId);
		}
	
		public CarBrand get(int itemId) {
			String sql = "SELECT * FROM brand WHERE idBrand=" + itemId;
			return jdbcTemplate.query(sql, new ResultSetExtractor<CarBrand>() {
	
				public CarBrand extractData(ResultSet rs) throws SQLException,
						DataAccessException {
					if (rs.next()) {
						return getCarBrandFromDB(rs);
					}
					return null;
				}
			});
		}
	
		public List<CarBrand> list() {
			String sql = "SELECT * FROM brand";
			List<CarBrand> listCarBrand = jdbcTemplate.query(sql, new RowMapper<CarBrand>() {
	
				public CarBrand mapRow(ResultSet rs, int i) throws SQLException {
	
					return getCarBrandFromDB(rs);
				}
			});
			return listCarBrand;
		}
	
		private CarBrand getCarBrandFromDB(ResultSet rs) throws SQLException{
			CarBrand brand = new CarBrand();
			brand.setIdBrand(rs.getInt("idBrand"));
			brand.setName(rs.getString("name"));
			brand.setFoundedYear(rs.getShort("foundedYear"));
			brand.setHeadquarter(rs.getString("headquarter"));
			return brand;
		}
	}

2. ``CarModelDAOJdbcImpl``:

.. code-block:: java
	:linenos:	
	
	package com.scrud.dao.jdbc;
	
	import com.scrud.dao.CarBrandDAO;
	import com.scrud.dao.CarModelDAO;
	import com.scrud.model.CarModel;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.dao.DataAccessException;
	import org.springframework.jdbc.core.JdbcTemplate;
	import org.springframework.jdbc.core.ResultSetExtractor;
	import org.springframework.jdbc.core.RowMapper;
	import javax.sql.DataSource;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.util.List;
	
	public class CarModelDAOJdbcImpl implements CarModelDAO {
		@Autowired
		@Qualifier("getCarBrandJdbcDAO")
		private CarBrandDAO carBrandDAO;
	
		private JdbcTemplate jdbcTemplate;
	
		public CarModelDAOJdbcImpl(DataSource dataSource) {
			jdbcTemplate = new JdbcTemplate(dataSource);
		}
	
		public void saveOrUpdate(CarModel item) {
			if (item.getIdModel() > 0) {
				// update
				String sql = "UPDATE model SET idBrand=?, modelName=?, generation=?, productionYear=?, doors=?, seats=?, maximumSpeed=? WHERE idModel=?";
				jdbcTemplate.update(sql, item.getIdBrand(), item.getModelName(),item.getGeneration(),
						item.getProductionYear(), item.getDoors(), item.getSeats(), item.getMaximumSpeed(), item.getIdModel());
			} else {
				// insert
				String sql = "INSERT INTO model (idBrand, modelName, generation, productionYear, doors, seats,maximumSpeed)"
						+ " VALUES (?, ?, ?, ?, ?, ?, ?)";
				jdbcTemplate.update(sql, item.getIdBrand(), item.getModelName(),item.getGeneration(),
						item.getProductionYear(), item.getDoors(), item.getSeats(), item.getMaximumSpeed());
			}
		}
	
		public void delete(int itemId) {
			String sql = "DELETE FROM model WHERE idModel=?";
			jdbcTemplate.update(sql, itemId);
		}
	
		public CarModel get(int itemId) {
			String sql = "SELECT * FROM model WHERE idModel=" + itemId;
			return jdbcTemplate.query(sql, new ResultSetExtractor<CarModel>() {
	
				public CarModel extractData(ResultSet rs) throws SQLException, DataAccessException {
					if (rs.next()) {
						return getCarModelFromDB(rs);
					}
					return null;
				}
			});
		}
	
		public List<CarModel> list() {
			String sql = "SELECT * FROM model";
			List<CarModel> listCarModel = jdbcTemplate.query(sql, new RowMapper<CarModel>() {
	
				public CarModel mapRow(ResultSet rs, int i) throws SQLException {
	
					return getCarModelFromDB(rs);
				}
			});
			return listCarModel;
		}
	
		private CarModel getCarModelFromDB(ResultSet rs) throws SQLException{
			CarModel model = new CarModel();
			model.setIdModel(rs.getInt("idModel"));
			model.setIdBrand(rs.getInt("idBrand"));
			model.setCarBrand(carBrandDAO.get(rs.getInt("idBrand")));
			model.setModelName(rs.getString("modelName"));
			model.setGeneration(rs.getString("generation"));
			model.setProductionYear(rs.getShort("productionYear"));
			model.setDoors(rs.getByte("doors"));
			model.setSeats(rs.getByte("seats"));
			model.setMaximumSpeed(rs.getShort("maximumSpeed"));
			return model;
		}
	}
	
Также и в пакете ``jpa`` создадим 2 класса:

1. ``CarBrandDAOJpaImpl``:

.. code-block:: java
	:linenos:
	
	package com.scrud.dao.jpa;
	
	import com.scrud.dao.CarBrandDAO;
	import com.scrud.model.CarBrand;
	import org.springframework.stereotype.Repository;
	import javax.persistence.EntityManager;
	import javax.persistence.PersistenceContext;
	import java.util.List;
	
	@Repository
	public class CarBrandDAOJpaImpl implements CarBrandDAO {
		@PersistenceContext
		private EntityManager em;
	
		public void saveOrUpdate(CarBrand item) {
			if (item.getIdBrand() > 0) {
				//update
				em.merge(item);
			} else {
				//insert
				em.persist(item);
			}
		}
	
		public void delete(int itemId) {
			em.remove(get(itemId));
	
		}
	
		public CarBrand get(int itemId) {
			return em.find(CarBrand.class, itemId);
		}
	
		public List<CarBrand> list() {
			List<CarBrand> carBrands = em.createNamedQuery("CarBrand.findAll",CarBrand.class).getResultList();
			return carBrands;
		}
	}	
	
2. ``CarModelDAOJpaImpl``:

.. code-block:: java
	:linenos:	
	
	package com.scrud.dao.jpa;
	
	import com.scrud.dao.CarModelDAO;
	import com.scrud.model.CarModel;
	import org.springframework.stereotype.Repository;
	import javax.persistence.EntityManager;
	import javax.persistence.PersistenceContext;
	import java.util.List;
	
	@Repository
	public class CarModelDAOJpaImpl implements CarModelDAO {
		@PersistenceContext
		private EntityManager em;
	
		public void saveOrUpdate(CarModel item) {
			if (item.getIdBrand() > 0) {
				// update
				em.merge(item);
			} else {
				// insert
				em.persist(item);
			}
		}
	
		public void delete(int itemId) {
			em.remove(get(itemId));
		}
	
		public CarModel get(int itemId) {
			return em.find(CarModel.class, itemId);
		}
	
		public List<CarModel> list() {
			List<CarModel> carModels = em.createNamedQuery("CarModel.findAll",CarModel.class).getResultList();
			return carModels;
		}
	}
	
Создание сервисов
---------------------

В пакете ``com.scrud`` создадим новый пакет ``service``, который будет содержать сервисы каждого варианта
осуществления запросов. Как и в случае с классами доступа к данным, сначала создадим интерфесы и потом
их реализацию для hibernate, jdbc и jpa.

В пакет ``service`` создадим интерфейсы сервисов, соответствующих ранее созданным сущностям:

1. ``CarBrandService``:

.. code-block:: java
	:linenos:
	
	package com.scrud.service;

	import com.scrud.model.CarBrand;
	import java.util.List;
	
	public interface CarBrandService {
	
		void saveOrUpdate(CarBrand item);
	
		void delete(int itemId);
	
		CarBrand get(int itemId);
	
		List<CarBrand> list();
	}
	
2. ``CarModelService``:

.. code-block:: java
	:linenos:
	
	package com.scrud.service;

	import com.scrud.model.CarModel;
	import java.util.List;
	
	public interface CarModelService {
		void saveOrUpdate(CarModel item);
	
		void delete(int itemId);
	
		CarModel get(int itemId);
	
		List<CarModel> list();
	}

Теперь реализуем интерфесы для hibernate. Создадим в пакете ``service`` 2 класса:

1. ``CarBrandHibernateServiceImpl``:

.. code-block:: java
	:linenos:
	
	package com.scrud.service;
	
	import com.scrud.dao.CarBrandDAO;
	import com.scrud.model.CarBrand;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.stereotype.Service;
	import org.springframework.transaction.annotation.Transactional;
	import java.util.List;
	
	@Service("carBrandHibernateService")
	@Transactional(readOnly=false, value = "hibernateTransactionManager")
	public class CarBrandHibernateServiceImpl implements CarBrandService {
		@Autowired
		@Qualifier("getCarBrandHibernateDAO")
		private CarBrandDAO carBrandDAO;
	
		public void saveOrUpdate(CarBrand item) {
			carBrandDAO.saveOrUpdate(item);
		}
	
		public void delete(int itemId) {
			carBrandDAO.delete(itemId);
		}
	
		public CarBrand get(int itemId) {
			return carBrandDAO.get(itemId);
		}
	
		public List<CarBrand> list() {
			return carBrandDAO.list();
		}
	}

2. ``CarModelHibernateServiceImpl``:

.. code-block:: java
	:linenos:

	package com.scrud.service;
	
	import com.scrud.dao.CarModelDAO;
	import com.scrud.model.CarModel;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.stereotype.Service;
	import org.springframework.transaction.annotation.Transactional;
	import java.util.List;
	
	@Service("carModelHibernateService")
	@Transactional(readOnly=false, value = "hibernateTransactionManager")
	public class CarModelHibernateServiceImpl implements CarModelService {
		@Autowired
		@Qualifier("getCarModelHibernateDAO")
		private CarModelDAO carModelDAO;
	
		public void saveOrUpdate(CarModel item) {
			carModelDAO.saveOrUpdate(item);
		}
	
		public void delete(int itemId) {
			carModelDAO.delete(itemId);
		}
	
		public CarModel get(int itemId) {
			return carModelDAO.get(itemId);
		}
	
		public List<CarModel> list() {
			return carModelDAO.list();
		}
	}

Для jdbc повторим аналогичную процедуру, создав в том же пакете классы:

1. ``CarBrandJdbcServiceImpl``:

.. code-block:: java
	:linenos:

	package com.scrud.service;
	
	import com.scrud.dao.CarBrandDAO;
	import com.scrud.model.CarBrand;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.stereotype.Service;
	import java.util.List;
	
	@Service("carBrandJdbcService")
	public class CarBrandJdbcServiceImpl implements CarBrandService {
		@Autowired
		@Qualifier("getCarBrandJdbcDAO")
		private CarBrandDAO carBrandDAO;
	
		public void saveOrUpdate(CarBrand item) {
			carBrandDAO.saveOrUpdate(item);
		}
	
		public void delete(int itemId) {
			carBrandDAO.delete(itemId);
		}
	
		public CarBrand get(int itemId) {
			return carBrandDAO.get(itemId);
		}
	
		public List<CarBrand> list() {
			return carBrandDAO.list();
		}
	}

2. ``CarModelJdbcService``:

.. code-block:: java
	:linenos:
	
	package com.scrud.service;
	
	import com.scrud.dao.CarModelDAO;
	import com.scrud.model.CarModel;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.stereotype.Service;
	import java.util.List;
	
	@Service("carModelJdbcService")
	public class CarModelJdbcServiceImpl implements CarModelService {
		@Autowired
		@Qualifier("getCarModelJdbcDAO")
		private CarModelDAO carModelDAO;
	
		public void saveOrUpdate(CarModel item) {
			carModelDAO.saveOrUpdate(item);
		}
	
		public void delete(int itemId) {
			carModelDAO.delete(itemId);
		}
	
		public CarModel get(int itemId) {
			return carModelDAO.get(itemId);
		}
	
		public List<CarModel> list() {
			return carModelDAO.list();
		}
	}

Ну и для jpa также в пакете ``service`` создадим 2 новых класса:

1. ``CarBrandJpaService``:

.. code-block:: java
	:linenos:
	
	package com.scrud.service;
	
	import com.scrud.dao.CarBrandDAO;
	import com.scrud.model.CarBrand;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.stereotype.Service;
	import org.springframework.transaction.annotation.Transactional;
	import java.util.List;
	
	@Service("carBrandJpaService")
	@Transactional(readOnly=false, value = "jpaTransactionManager")
	public class CarBrandJpaServiceImpl implements CarBrandService {
		@Autowired
		@Qualifier("getCarBrandJpaDAO")
		private CarBrandDAO carBrandDAO;
	
		public List<CarBrand> list() {
			return carBrandDAO.list();
		}
	
		public CarBrand get(int itemId) {
			return carBrandDAO.get(itemId);
		}
	
		public void saveOrUpdate(CarBrand item) {
			carBrandDAO.saveOrUpdate(item);
		}
	
		public void delete(int itemId) {
			carBrandDAO.delete(itemId);
		}
	}
	
2. ``CarModelJpaService``:

.. code-block:: java
	:linenos:
	
	package com.scrud.service;
	
	import com.scrud.dao.CarModelDAO;
	import com.scrud.model.CarModel;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.stereotype.Service;
	import org.springframework.transaction.annotation.Transactional;
	import java.util.List;
	
	@Service("carModelJpaService")
	@Transactional(readOnly=false, value = "jpaTransactionManager")
	public class CarModelJpaServiceImpl implements CarModelService {
		@Autowired
		@Qualifier("getCarModelJpaDAO")
		private CarModelDAO carModelDAO;
	
		public void saveOrUpdate(CarModel item) {
			carModelDAO.saveOrUpdate(item);
		}
	
		public void delete(int itemId) {
			carModelDAO.delete(itemId);
		}
	
		public CarModel get(int itemId) {
			return carModelDAO.get(itemId);
		}
	
		public List<CarModel> list() {
			return carModelDAO.list();
		}
	}
	
Как видите, все сервисы очень похожи, но мы специально разделили их в разные классы для наглядной
демонстрации трех различных вариантов реализации.

Вывод данных в файл
---------------------

Нам необходимо реализовать вывод данных в файл PDF и Excel форматов.

В директории **main** -> **resources** создадим файл ``view.properties`` и запишем в него
конфигурацию классов формирования файлов:
::

	pdfView.(class)=com.scrud.view.PDFBuilder
	excelView.(class)=com.scrud.view.ExcelBuilder

Далее в пакете ``com.scrud`` создадим новый пакет ``view`` и поместим в него следующие классы:

1. ``AbstractTextPdfView``:

.. code-block:: java
	:linenos:
	
	package com.scrud.view;
	
	import java.io.ByteArrayOutputStream;
	import java.io.OutputStream;
	import java.text.SimpleDateFormat;
	import java.util.Date;
	import java.util.Map;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import com.itextpdf.text.*;
	import com.itextpdf.text.pdf.ColumnText;
	import com.itextpdf.text.pdf.PdfPageEventHelper;
	import org.springframework.web.servlet.view.AbstractView;
	import com.itextpdf.text.pdf.PdfWriter;
	
	public abstract class AbstractITextPdfView extends AbstractView {
	
		public AbstractITextPdfView() {
			setContentType("application/pdf");
		}
	
		@Override
		protected boolean generatesDownloadContent() {
			return true;
		}
			
		@Override
		protected void renderMergedOutputModel(Map<String, Object> model,
				HttpServletRequest request, HttpServletResponse response) throws Exception {
			// IE workaround: write into byte array first.
			ByteArrayOutputStream baos = createTemporaryOutputStream();
	
			// Apply preferences and build metadata.
			Document document = newDocument();
			PdfWriter writer = newWriter(document, baos);
			prepareWriter(model, writer, request);
			buildPdfMetadata(model, document, request);
	
			// Build PDF document.
			document.open();
			addMetaData(document);
			buildPdfDocument(model, document, writer, request, response);
			document.close();
	
			// Flush to HTTP response.
			writeToResponse(response, baos);
		}
	
		private static void addMetaData(Document document) {
	
		}
	
		protected Document newDocument() {
			return new Document(PageSize.A4);
		}
		
		protected PdfWriter newWriter(Document document, OutputStream os) throws DocumentException {
			return PdfWriter.getInstance(document, os);
		}
		
		protected void prepareWriter(Map<String, Object> model, PdfWriter writer, HttpServletRequest request) throws DocumentException {
	
			FooterPageEvent event = new FooterPageEvent();
			writer.setPageEvent(event);
			writer.setViewerPreferences(getViewerPreferences());
		}
		
		protected int getViewerPreferences() {
			return PdfWriter.ALLOW_PRINTING | PdfWriter.PageLayoutSinglePage;
		}
		
		protected void buildPdfMetadata(Map<String, Object> model, Document document, HttpServletRequest request) {
			document.addTitle("Car_report");
			document.addSubject("Using iText with Spring");
			document.addKeywords("Java, PDF, iText");
			document.addAuthor("mchukd@gmail.com");
			document.addCreator("mchukd@gmail.com");
		}
		
		protected abstract void buildPdfDocument(Map<String, Object> model, Document document, PdfWriter writer,
				HttpServletRequest request, HttpServletResponse response) throws Exception;
	
		public class FooterPageEvent extends PdfPageEventHelper {
	
			private Font font = FontFactory.getFont(FontFactory.COURIER);
	
			public void onEndPage(PdfWriter writer, Document document) {
	
				SimpleDateFormat formatter = new SimpleDateFormat("dd.MM.yyyy 'at' HH:mm:ss");
				String today = formatter.format(new Date());
	
				ColumnText.showTextAligned(writer.getDirectContent(), Element.ALIGN_CENTER, new Phrase("generated: "+today, font), 290, 40, 0);
			}
		}
	}
	
2. ``AbstractPOIExcelView``:

.. code-block:: java
	:linenos:
	
	package com.scrud.view;
	
	import org.apache.poi.POIXMLProperties;
	import org.apache.poi.xssf.usermodel.XSSFWorkbook;
	import org.springframework.web.servlet.view.AbstractView;
	import javax.servlet.ServletOutputStream;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import java.util.Map;
	
	public abstract class AbstractPOIExcelView extends AbstractView{
	
		private static final String CONTENT_TYPE_XLSX = "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
	
		public AbstractPOIExcelView() {
			setContentType(CONTENT_TYPE_XLSX);
		}
	
		@Override
		protected boolean generatesDownloadContent() {
			return true;
		}
	
		@Override
		protected final void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request,
													HttpServletResponse response) throws Exception {
	
			XSSFWorkbook workbook = createWorkbook();
			buildXlsxMetadata(workbook);
			buildExcelDocument(model, workbook, request, response);
			// Set the content type.
			response.setContentType(getContentType());
	
			// Flush byte array to servlet output stream.
			ServletOutputStream out = response.getOutputStream();
			out.flush();
			workbook.write(out);
			out.flush();
		}
		protected void buildXlsxMetadata(XSSFWorkbook workbook){
			POIXMLProperties props = workbook.getProperties();
	
			POIXMLProperties.CoreProperties coreProp=props.getCoreProperties();
			coreProp.setCreator("mchukd@gmail.com");
			coreProp.setDescription("creating xlsx report using Apache POI / Java");
			coreProp.setKeywords("Apache POI, Java, Spring, XLSX ");
			coreProp.setTitle("Cars_report");
			coreProp.setCategory("Programming");
	
			POIXMLProperties.ExtendedProperties extProp=props.getExtendedProperties();
			extProp.getUnderlyingProperties().setCompany("knit-12-1");
			extProp.getUnderlyingProperties().setManager("mk_d");
		}
	
		protected abstract XSSFWorkbook createWorkbook();
	
		protected abstract void buildExcelDocument(Map<String, Object> model, XSSFWorkbook workbook,
												HttpServletRequest request, HttpServletResponse response) throws Exception;
	}
	
3. ``ExcelBuilder``:

.. code-block:: java
	:linenos:

	package com.scrud.view;
	
	import java.util.List;
	import java.util.Map;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import com.scrud.model.CarBrand;
	import com.scrud.model.CarModel;
	import org.apache.poi.hssf.usermodel.HSSFFont;
	import org.apache.poi.hssf.util.HSSFColor;
	import org.apache.poi.ss.usermodel.*;
	import org.apache.poi.xssf.usermodel.XSSFWorkbook;
	
	public class ExcelBuilder extends AbstractPOIExcelView {
		@Override
		protected XSSFWorkbook createWorkbook() {
			return new XSSFWorkbook();
		}
	
		@Override
		protected void buildExcelDocument(Map<String, Object> model, XSSFWorkbook workbook, HttpServletRequest request, HttpServletResponse response) throws Exception {
			// get data model which is passed by the Spring container
			List<CarBrand> carBrands = (List<CarBrand>) model.get("carBrands");
			List<CarModel> carModels = (List<CarModel>) model.get("carModels");
	
			generateBrandSheet(workbook, carBrands);
			generateModelsSheet(workbook, carModels);
		}
	
		private void generateBrandSheet(XSSFWorkbook workbook, List<CarBrand> carBrands){
	
			// create a new Excel sheet
			Sheet sheet = workbook.createSheet("Brands");
			sheet.setColumnWidth(0,2000);
			sheet.setColumnWidth(1,5000);
			sheet.setColumnWidth(2,3000);
			sheet.setColumnWidth(3,10000);
	
			Font font = workbook.createFont();
			font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);
			font.setColor(HSSFColor.WHITE.index);
			font.setFontName("Helvetica");
	
			CellStyle headerStyle = workbook.createCellStyle();
			headerStyle.setFillForegroundColor(HSSFColor.GREY_40_PERCENT.index);
			headerStyle.setFillPattern(CellStyle.SOLID_FOREGROUND);
			headerStyle.setFont(font);
			headerStyle.setAlignment(CellStyle.ALIGN_CENTER);
	
			// create header row
			Row headerRow = sheet.createRow(0);
			headerRow.createCell(0).setCellValue("№");
			headerRow.getCell(0).setCellStyle(headerStyle);
	
			headerRow.createCell(1).setCellValue("Brand name");
			headerRow.getCell(1).setCellStyle(headerStyle);
	
			headerRow.createCell(2).setCellValue("Founded year");
			headerRow.getCell(2).setCellStyle(headerStyle);
	
			headerRow.createCell(3).setCellValue("Headquarter");
			headerRow.getCell(3).setCellStyle(headerStyle);
	
			CellStyle rowCellStyle = workbook.createCellStyle();
			rowCellStyle.setAlignment(CellStyle.ALIGN_CENTER);
			// create data rows
			int rowCount = 1;
			for (CarBrand carBrand : carBrands) {
				Row row = sheet.createRow(rowCount++);
				row.createCell(0).setCellValue(rowCount-1);
				row.getCell(0).setCellStyle(rowCellStyle);
	
				row.createCell(1).setCellValue(carBrand.getName());
				row.getCell(1).setCellStyle(rowCellStyle);
	
				row.createCell(2).setCellValue(carBrand.getFoundedYear());
				row.getCell(2).setCellStyle(rowCellStyle);
	
				row.createCell(3).setCellValue(carBrand.getHeadquarter());
				row.getCell(3).setCellStyle(rowCellStyle);
			}
		}
	
		private void generateModelsSheet(XSSFWorkbook workbook, List<CarModel> carModels){
	
			// create a new Excel sheet
			Sheet sheet = workbook.createSheet("Models");
			sheet.setColumnWidth(0,2000);
			sheet.setColumnWidth(1,5000);
			sheet.setColumnWidth(2,5000);
			sheet.setColumnWidth(3,7000);
			sheet.setColumnWidth(4,2500);
			sheet.setColumnWidth(5,2000);
			sheet.setColumnWidth(6,2000);
			sheet.setColumnWidth(7,2500);
	
			Font font = workbook.createFont();
			font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);
			font.setColor(HSSFColor.WHITE.index);
			font.setFontName("Helvetica");
	
			CellStyle headerStyle = workbook.createCellStyle();
			headerStyle.setFillForegroundColor(HSSFColor.GREY_40_PERCENT.index);
			headerStyle.setFillPattern(CellStyle.SOLID_FOREGROUND);
			headerStyle.setFont(font);
			headerStyle.setAlignment(CellStyle.ALIGN_CENTER);
	
			// create header row
			Row headerRow = sheet.createRow(0);
			headerRow.createCell(0).setCellValue("№");
			headerRow.getCell(0).setCellStyle(headerStyle);
	
			headerRow.createCell(1).setCellValue("Brand");
			headerRow.getCell(1).setCellStyle(headerStyle);
	
			headerRow.createCell(2).setCellValue("Model");
			headerRow.getCell(2).setCellStyle(headerStyle);
	
			headerRow.createCell(3).setCellValue("Generation");
			headerRow.getCell(3).setCellStyle(headerStyle);
	
			headerRow.createCell(4).setCellValue("Prod year");
			headerRow.getCell(4).setCellStyle(headerStyle);
	
			headerRow.createCell(5).setCellValue("Doors");
			headerRow.getCell(5).setCellStyle(headerStyle);
	
			headerRow.createCell(6).setCellValue("Seats");
			headerRow.getCell(6).setCellStyle(headerStyle);
	
			headerRow.createCell(7).setCellValue("Max speed");
			headerRow.getCell(7).setCellStyle(headerStyle);
	
			CellStyle rowCellStyle = workbook.createCellStyle();
			rowCellStyle.setAlignment(CellStyle.ALIGN_CENTER);
			// create data rows
			int rowCount = 1;
			for (CarModel carModel : carModels) {
				Row row = sheet.createRow(rowCount++);
				row.createCell(0).setCellValue(rowCount-1);
				row.getCell(0).setCellStyle(rowCellStyle);
	
				row.createCell(1).setCellValue(carModel.getCarBrand().getName());
				row.getCell(1).setCellStyle(rowCellStyle);
	
				row.createCell(2).setCellValue(carModel.getModelName());
				row.getCell(2).setCellStyle(rowCellStyle);
	
				row.createCell(3).setCellValue(carModel.getGeneration());
				row.getCell(3).setCellStyle(rowCellStyle);
	
				row.createCell(4).setCellValue(carModel.getProductionYear());
				row.getCell(4).setCellStyle(rowCellStyle);
	
				row.createCell(5).setCellValue(carModel.getDoors());
				row.getCell(5).setCellStyle(rowCellStyle);
	
				row.createCell(6).setCellValue(carModel.getSeats());
				row.getCell(6).setCellStyle(rowCellStyle);
	
				row.createCell(7).setCellValue(carModel.getMaximumSpeed());
				row.getCell(7).setCellStyle(rowCellStyle);
			}
		}
	
	}
	
4. ``PDFBuilder``:

.. code-block:: java
	:linenos:
	
	package com.scrud.view;
	
	import java.util.List;
	import java.util.Map;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import com.itextpdf.text.*;
	import com.itextpdf.text.pdf.PdfPCell;
	import com.itextpdf.text.pdf.PdfPTable;
	import com.itextpdf.text.pdf.PdfWriter;
	import com.scrud.model.CarBrand;
	import com.scrud.model.CarModel;
	
	public class PDFBuilder extends AbstractITextPdfView {
	
		@Override
		protected void buildPdfDocument(Map<String, Object> model, Document doc,
										PdfWriter writer, HttpServletRequest request,
										HttpServletResponse response)throws Exception {
	
			// get data model which is passed by the Spring container
			Font paragraphFont = FontFactory.getFont(FontFactory.HELVETICA_BOLD);
			paragraphFont.setSize(14);
	
			List<CarBrand> carBrands = (List<CarBrand>) model.get("carBrands");
			Paragraph brandParagraph = new Paragraph("Car brands",paragraphFont);
			brandParagraph.setAlignment(Element.ALIGN_CENTER);
			doc.add(brandParagraph);
			PdfPTable carBrandsTable = getCarBrandTable(carBrands);
			doc.add(carBrandsTable);
	
			List<CarModel> carModels = (List<CarModel>) model.get("carModels");
			Paragraph modelParagraph = new Paragraph("Car models",paragraphFont);
			modelParagraph.setAlignment(Element.ALIGN_CENTER);
			doc.add(modelParagraph);
			PdfPTable carModelsTable = getCarModelTable(carModels);
			doc.add(carModelsTable);
		}
	
	
		private PdfPTable getCarBrandTable(List<CarBrand> carBrands) throws Exception {
	
			PdfPTable table = new PdfPTable(4);
			table.setWidthPercentage(100.0f);
			table.setWidths(new float[]{0.4f, 2.0f, 1.0f, 4.0f});
			table.setSpacingBefore(10);
			table.setHorizontalAlignment(Element.ALIGN_RIGHT);
			// define font for table header row
			Font font = FontFactory.getFont(FontFactory.HELVETICA, "ISO-8859-5");
			font.setColor(BaseColor.WHITE);
	
			// define table header cell
			PdfPCell cell = new PdfPCell();
	
			cell.setBackgroundColor(BaseColor.GRAY);
			cell.setPadding(5);
			cell.setHorizontalAlignment(Element.ALIGN_CENTER);
	
			// write table header
			cell.setPhrase(new Phrase("#", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Brand name", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Founded year", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Headquarter", font));
			table.addCell(cell);
	
			// write table row data
			int index = 1;
			for (CarBrand carBrand : carBrands) {
				table.addCell("" + index);
				table.addCell(carBrand.getName());
				table.addCell("" + carBrand.getFoundedYear());
				table.addCell(carBrand.getHeadquarter());
				index++;
			}
			return table;
		}
	
		private PdfPTable getCarModelTable(List<CarModel> carModels) throws Exception {
	
			PdfPTable table = new PdfPTable(8);
			table.setWidthPercentage(100.0f);
			table.setWidths(new float[]{0.4f, 1.5f, 1.5f, 2.0f, 0.7f, 0.7f, 0.7f, 0.7f});
			table.setSpacingBefore(5);
	
			// define font for table header row
			Font font = FontFactory.getFont(FontFactory.HELVETICA, "ISO-8859-5");
			font.setColor(BaseColor.WHITE);
	
			// define table header cell
			PdfPCell cell = new PdfPCell();
			cell.setBackgroundColor(BaseColor.GRAY);
			cell.setPadding(5);
			cell.setHorizontalAlignment(Element.ALIGN_CENTER);
	
			// write table header
			cell.setPhrase(new Phrase("#", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Brand", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Model", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Generation", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Prod year", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Doors", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Seats", font));
			table.addCell(cell);
	
			cell.setPhrase(new Phrase("Max speed", font));
			table.addCell(cell);
	
			int index = 1;
			// write table row data
			for (CarModel carModel : carModels) {
				table.addCell("" + index);
				table.addCell(carModel.getCarBrand().getName());
				table.addCell(carModel.getModelName());
				table.addCell(carModel.getGeneration());
				table.addCell("" + carModel.getProductionYear());
				table.addCell("" + carModel.getDoors());
				table.addCell("" + carModel.getSeats());
				table.addCell("" + carModel.getMaximumSpeed());
				index++;
			}
			return table;
		}
	}
	

Создание контроллеров
----------------------

Теперь для hibernate, jdbc и  jpa создадим контроллеры, которые будут исопльзовать добавленные ранее сервисы
а также вывод данных файл.Для этого в пакете ``controllers`` создадим 3 новых класса:

1. ``HibernateController``:

.. code-block:: java
	:linenos:
	
	package com.scrud.controllers;
	
	import com.scrud.model.CarBrand;
	import com.scrud.model.CarModel;
	import com.scrud.service.CarBrandService;
	import com.scrud.service.CarModelService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.stereotype.Controller;
	import org.springframework.ui.ModelMap;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.servlet.ModelAndView;
	import java.util.List;
	
	@Controller
	public class HibernateController {
	
		private static final String INSTRUMENT = "hibernate";
		private static final String TITLE = "Hibernate";
	
		@Autowired
		@Qualifier("carBrandHibernateService")
		private CarBrandService carBrandService;
		@Autowired
		@Qualifier("carModelHibernateService")
		private CarModelService carModelService;
	
		@RequestMapping(value = "/"+INSTRUMENT+"", method = RequestMethod.GET)
		public String printJdbc(ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("instrument", INSTRUMENT);

			List<CarBrand> listCarBrand = carBrandService.list();
			List<CarModel> listCarModel = carModelService.list();
	
			model.addAttribute("listCarModel",listCarModel);
			model.addAttribute("listCarBrand",listCarBrand);
			return "content";
		}
	
		//CRUD oparations with CarBrand entity
		@RequestMapping(value = "/"+INSTRUMENT+"/newBrand", method = RequestMethod.GET)
		public String addBrand(ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Add new");
	
			CarBrand carBrand = new CarBrand();
			model.addAttribute("carBrand", carBrand);
	
			return "brandForm";
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/newBrand" }, method = RequestMethod.POST)
		public String saveBrand(CarBrand carBrand) {
	
			carBrandService.saveOrUpdate(carBrand);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/delete-brand/{idBrand}" }, method = RequestMethod.GET)
		public String deleteBrand(@PathVariable int idBrand) {
			carBrandService.delete(idBrand);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-brand/{idBrand}" }, method = RequestMethod.GET)
		public String editBrand(@PathVariable int idBrand, ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Edit");
	
			CarBrand carBrand = carBrandService.get(idBrand);
			model.addAttribute("carBrand", carBrand);
			model.addAttribute("edit", true);
			return "brandForm";
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-brand/{idBrand}" }, method = RequestMethod.POST)
		public String updateBrand(CarBrand carBrand) {
			carBrandService.saveOrUpdate(carBrand);
			return "redirect:/"+INSTRUMENT;
		}
	
		//CRUD oparations with CarModel entity
		@RequestMapping(value = "/"+INSTRUMENT+"/newModel", method = RequestMethod.GET)
		public String addModel(ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Add new");
	
			List<CarBrand> listCarBrand = carBrandService.list();
			System.out.println(listCarBrand);
			CarModel carModel = new CarModel();
			model.addAttribute("listCarBrand", listCarBrand);
			model.addAttribute("carModel", carModel);
	
			return "modelForm";
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/newModel" }, method = RequestMethod.POST)
		public String saveModel(CarModel carModel) {
	
			int idBrand = carModel.getIdBrand();
			carModel.setCarBrand(carBrandService.get(idBrand));
			carModelService.saveOrUpdate(carModel);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/delete-model/{idModel}" }, method = RequestMethod.GET)
		public String deleteUser(@PathVariable int idModel) {
			carModelService.delete(idModel);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-model/{idModel}" }, method = RequestMethod.GET)
		public String editModel(@PathVariable int idModel, ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Edit");
	
			CarModel carModel = carModelService.get(idModel);
			List<CarBrand> listCarBrand = carBrandService.list();
	
			model.addAttribute("carModel", carModel);
			model.addAttribute("listCarBrand", listCarBrand);
	
			return "modelForm";
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-model/{idModel}" }, method = RequestMethod.POST)
		public String updateModel(CarModel carModel) {
	
			int idBrand = carModel.getIdBrand();
			carModel.setCarBrand(carBrandService.get(idBrand));
	
			carModelService.saveOrUpdate(carModel);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = {"/"+INSTRUMENT+"/pdfReport", "/"+INSTRUMENT+"/xlsxReport.xlsx"}, method = RequestMethod.GET)
		public ModelAndView downloadReport(@RequestParam("view") String view) {
			ModelAndView modelAndView = new ModelAndView();
	
			List<CarBrand> listCarBrand = carBrandService.list();
			List<CarModel> listCarModel = carModelService.list();
			// return a view which will be resolved by an ResourceBundleViewResolver
			modelAndView.addObject("carBrands", listCarBrand);
			modelAndView.addObject("carModels", listCarModel);
			modelAndView.setViewName(view);
	
			return modelAndView;
		}
	}
	
2. ``JdbcController``:

.. code-block:: java
	:linenos:
	
	package com.scrud.controllers;
	
	import com.scrud.model.CarBrand;
	import com.scrud.model.CarModel;
	import com.scrud.service.CarBrandService;
	import com.scrud.service.CarModelService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.stereotype.Controller;
	import org.springframework.ui.ModelMap;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.servlet.ModelAndView;
	import java.util.*;
	
	@Controller
	public class JdbcController {
	
		private static final String INSTRUMENT = "jdbc";
		private static final String TITLE = "JDBC";
		
		@Autowired
		@Qualifier("carBrandJdbcService")
		private CarBrandService carBrandService;
		@Autowired
		@Qualifier("carModelJdbcService")
		private CarModelService carModelService;
	
		@RequestMapping(value = "/"+INSTRUMENT+"", method = RequestMethod.GET)
		public String printJdbc(ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("instrument", INSTRUMENT);
	
			List<CarBrand> listCarBrand = carBrandService.list();
			List<CarModel> listCarModel = carModelService.list();
	
			model.addAttribute("listCarModel",listCarModel);
			model.addAttribute("listCarBrand",listCarBrand);
			return "content";
		}
	
		//CRUD oparations with CarBrand entity
		@RequestMapping(value = "/"+INSTRUMENT+"/newBrand", method = RequestMethod.GET)
		public String addBrand(ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Add new");
	
			CarBrand carBrand = new CarBrand();
			model.addAttribute("carBrand", carBrand);
	
			return "brandForm";
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/newBrand" }, method = RequestMethod.POST)
		public String saveBrand(CarBrand carBrand) {
	
			carBrandService.saveOrUpdate(carBrand);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/delete-brand/{idBrand}" }, method = RequestMethod.GET)
		public String deleteBrand(@PathVariable int idBrand) {
			carBrandService.delete(idBrand);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-brand/{idBrand}" }, method = RequestMethod.GET)
		public String editBrand(@PathVariable int idBrand, ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Edit");
	
			CarBrand carBrand = carBrandService.get(idBrand);
			model.addAttribute("carBrand", carBrand);
			return "brandForm";
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-brand/{idBrand}" }, method = RequestMethod.POST)
		public String updateBrand(CarBrand carBrand) {
			carBrandService.saveOrUpdate(carBrand);
			return "redirect:/"+INSTRUMENT;
		}
	
		//CRUD oparations with CarModel entity
		@RequestMapping(value = "/"+INSTRUMENT+"/newModel", method = RequestMethod.GET)
		public String addModel(ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Add new");
	
			List<CarBrand> listCarBrand = carBrandService.list();
			System.out.println(listCarBrand);
			CarModel carModel = new CarModel();
			model.addAttribute("listCarBrand", listCarBrand);
			model.addAttribute("carModel", carModel);
	
			return "modelForm";
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/newModel" }, method = RequestMethod.POST)
		public String saveModel(CarModel carModel) {
	
			carModelService.saveOrUpdate(carModel);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/delete-model/{idModel}" }, method = RequestMethod.GET)
		public String deleteUser(@PathVariable int idModel) {
			carModelService.delete(idModel);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-model/{idModel}" }, method = RequestMethod.GET)
		public String editModel(@PathVariable int idModel, ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Edit");
	
			CarModel carModel = carModelService.get(idModel);
			List<CarBrand> listCarBrand = carBrandService.list();
	
			model.addAttribute("carModel", carModel);
			model.addAttribute("listCarBrand", listCarBrand);
	
			return "modelForm";
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-model/{idModel}" }, method = RequestMethod.POST)
		public String updateModel(CarModel carModel) {
			carModelService.saveOrUpdate(carModel);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = {"/"+INSTRUMENT+"/pdfReport", "/"+INSTRUMENT+"/xlsxReport.xlsx"}, method = RequestMethod.GET)
		public ModelAndView downloadReport(@RequestParam("view") String view) {
			ModelAndView modelAndView = new ModelAndView();
	
			List<CarBrand> listCarBrand = carBrandService.list();
			List<CarModel> listCarModel = carModelService.list();
			// return a view which will be resolved by an ResourceBundleViewResolver
			modelAndView.addObject("carBrands", listCarBrand);
			modelAndView.addObject("carModels", listCarModel);
			modelAndView.setViewName(view);
	
			return modelAndView;
		}
	}

3. ``JpaController``:

.. code-block:: java
	:linenos:
	
	package com.scrud.controllers;
	
	import com.scrud.model.CarBrand;
	import com.scrud.model.CarModel;
	import com.scrud.service.CarBrandService;
	import com.scrud.service.CarModelService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.stereotype.Controller;
	import org.springframework.ui.ModelMap;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.servlet.ModelAndView;
	import java.util.List;
	
	@Controller
	public class JpaController {
		
		private static final String INSTRUMENT = "jpa";
		private static final String TITLE = "JPA";
		
		@Autowired
		@Qualifier("carBrandJpaService")
		private CarBrandService carBrandService;
		@Autowired
		@Qualifier("carModelJpaService")
		private CarModelService carModelService;
	
		@RequestMapping(value = "/"+INSTRUMENT+"", method = RequestMethod.GET)
		public String printJdbc(ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("instrument", INSTRUMENT);
	
			List<CarBrand> listCarBrand = carBrandService.list();
			List<CarModel> listCarModel = carModelService.list();
	
			model.addAttribute("listCarModel",listCarModel);
			model.addAttribute("listCarBrand",listCarBrand);
	
			return "content";
		}
	
		//CRUD oparations with CarBrand entity
		@RequestMapping(value = "/"+INSTRUMENT+"/newBrand", method = RequestMethod.GET)
		public String addBrand(ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Add new");
	
			CarBrand carBrand = new CarBrand();
			model.addAttribute("carBrand", carBrand);
	
			return "brandForm";
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/newBrand" }, method = RequestMethod.POST)
		public String saveBrand(CarBrand carBrand) {
	
			carBrandService.saveOrUpdate(carBrand);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/delete-brand/{idBrand}" }, method = RequestMethod.GET)
		public String deleteBrand(@PathVariable int idBrand) {
			carBrandService.delete(idBrand);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-brand/{idBrand}" }, method = RequestMethod.GET)
		public String editBrand(@PathVariable int idBrand, ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Edit");
	
			CarBrand carBrand = carBrandService.get(idBrand);
			model.addAttribute("carBrand", carBrand);
			model.addAttribute("edit", true);
			return "brandForm";
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-brand/{idBrand}" }, method = RequestMethod.POST)
		public String updateBrand(CarBrand carBrand) {
			carBrandService.saveOrUpdate(carBrand);
			return "redirect:/"+INSTRUMENT;
		}
	
		//CRUD oparations with CarModel entity
		@RequestMapping(value = "/"+INSTRUMENT+"/newModel", method = RequestMethod.GET)
		public String addModel(ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Add new");
	
			List<CarBrand> listCarBrand = carBrandService.list();
			System.out.println(listCarBrand);
			CarModel carModel = new CarModel();
			model.addAttribute("listCarBrand", listCarBrand);
			model.addAttribute("carModel", carModel);
	
			return "modelForm";
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/newModel" }, method = RequestMethod.POST)
		public String saveModel(CarModel carModel) {
	
			int idBrand = carModel.getIdBrand();
			carModel.setCarBrand(carBrandService.get(idBrand));
			carModelService.saveOrUpdate(carModel);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = { "/"+INSTRUMENT+"/delete-model/{idModel}" }, method = RequestMethod.GET)
		public String deleteUser(@PathVariable int idModel) {
			carModelService.delete(idModel);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-model/{idModel}" }, method = RequestMethod.GET)
		public String editModel(@PathVariable int idModel, ModelMap model) {
			model.addAttribute("title", TITLE);
			model.addAttribute("action", "Edit");
	
			CarModel carModel = carModelService.get(idModel);
			List<CarBrand> listCarBrand = carBrandService.list();
	
			model.addAttribute("carModel", carModel);
			model.addAttribute("listCarBrand", listCarBrand);
	
			return "modelForm";
		}
	
		@RequestMapping(value = {  "/"+INSTRUMENT+"/edit-model/{idModel}" }, method = RequestMethod.POST)
		public String updateModel(CarModel carModel) {
	
			int idBrand = carModel.getIdBrand();
			carModel.setCarBrand(carBrandService.get(idBrand));
	
			carModelService.saveOrUpdate(carModel);
			return "redirect:/"+INSTRUMENT;
		}
	
		@RequestMapping(value = {"/"+INSTRUMENT+"/pdfReport", "/"+INSTRUMENT+"/xlsxReport.xlsx"}, method = RequestMethod.GET)
		public ModelAndView downloadReport(@RequestParam("view") String view) {
			ModelAndView modelAndView = new ModelAndView();
	
			List<CarBrand> listCarBrand = carBrandService.list();
			List<CarModel> listCarModel = carModelService.list();
			// return a view which will be resolved by an ResourceBundleViewResolver
			modelAndView.addObject("carBrands", listCarBrand);
			modelAndView.addObject("carModels", listCarModel);
			modelAndView.setViewName(view);
	
			return modelAndView;
		}
	}
	
Опять же, все 3 класса идентичны и отличаются только переменными, соответствующими их названиям. 
Это сделано для независимости реализаций каждого варианта доступа.



`Скачаь ресурсы <https://github.com/Xen4n/CRUD_docs/blob/master/docs/_static/resources.7z?raw=true>`_
