.. _frontend:

Создание страниц и оформления
==============================

Наш сайт почти готов. Осталось только создать страницы, которые будут возвращаться пользователю
после запроса на сервер. 

Создание страниц
-----------------

Мы уже создавали главную страницу сайта. Теперь в ту же дерикторию ``pages`` добавим 3 новых файла JSP:

1. ``brandForm.jsp``:
::
	
	<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
	<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
	<head>
		<link href="<c:url value="${pageContext.request.contextPath}/webjars/bootstrap/3.1.0/css/bootstrap.min.css" />" rel="stylesheet">
		<link href="/resources/css/basic.css" rel="stylesheet">
	
		<script src="<c:url value="${pageContext.request.contextPath}/webjars/jquery/1.9.0/jquery.min.js"  />"></script>
		<script src="<c:url value="${pageContext.request.contextPath}/webjars/bootstrap/3.1.0/js/bootstrap.js"  />"></script>
	
		<title>${title}</title>
	</head>
	<body>
		<c:import url="page_components/header.jsp"></c:import>
		<div class="container" >
		<div class="row">
			<div class="col-lg-6 col-lg-offset-3">
				<div class="panel panel-primary">
					<div class="panel-heading">
						<div class="text-center">
							<h1>${action} brand <small>using ${title}</small></h1>
						</div>
					</div>
					<div class="panel-body">
						<form:form method="POST" modelAttribute="carBrand" class="form-horizontal">
							<form:hidden path="idBrand"/>
							<div class="form-group">
								<label for="name" class="col-sm-3 control-label">Brand name:</label>
								<div class="col-sm-9">
									<form:input path="name" class="form-control" />
								</div>
							</div>
							<div class="form-group">
								<label for="foundedYear" class="col-sm-3 control-label">Founded year:</label>
								<div class="col-sm-9">
									<form:input path="foundedYear" class="form-control" type="number" min="1800" max="2050"  />
								</div>
							</div>
							<div class="form-group">
								<label for="headquarter" class="col-sm-3 control-label">Headquarter:</label>
								<div class="col-sm-9">
									<form:input path="headquarter" class="form-control" />
								</div>
							</div>
							<div class="form-group">
								<div class="col-sm-offset-3 col-sm-9">
									<button type="submit" class="btn btn-primary">Save</button>
								</div>
							</div>
						</form:form>
					</div>
					</div>
				</div>
			</div>
		</div>
	</body>
	</html>
	
2. ``modelForm.jsp``:
::

	<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
	<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
	<head>
		<meta name="viewport" content="initial-scale=1, maximum-scale=1">
		<link href="<c:url value="${pageContext.request.contextPath}/webjars/bootstrap/3.1.0/css/bootstrap.min.css" />" rel="stylesheet">
		<link href="/resources/css/basic.css" rel="stylesheet">
	
		<script src="<c:url value="${pageContext.request.contextPath}/webjars/jquery/1.9.0/jquery.min.js"  />"></script>
		<script src="<c:url value="${pageContext.request.contextPath}/webjars/bootstrap/3.1.0/js/bootstrap.js"  />"></script>
	
		<title>${title}</title>
	</head>
	<body>
	<c:import url="page_components/header.jsp"></c:import>
	<div class="container" >
		<div class="row">
			<div class="col-lg-6 col-lg-offset-3">
				<div class="panel panel-primary">
					<div class="panel-heading">
						<div class="text-center">
							<h1>${action} model <small>using ${title}</small></h1>
						</div>
					</div>
					<div class="panel-body">
						<form:form method="POST" modelAttribute="carModel" class="form-horizontal">
							<form:hidden path="idModel"/>
							<div class="form-group">
								<label for="idBrand" class="col-sm-3 control-label" >Brand:</label>
								<div class="col-sm-9">
									<form:select path="idBrand" multiple="false" class="form-control">
										<c:forEach var="brand" items="${listCarBrand}" varStatus="status">
											<c:choose>
												<c:when test="${brand.idBrand == carModel.idBrand}">
													<option selected value="${brand.idBrand}">${brand.name}</option>
												</c:when>
												<c:otherwise>
													<option value="${brand.idBrand}">${brand.name}</option>
												</c:otherwise>
											</c:choose>
										</c:forEach>
									</form:select>
								</div>
							</div>
							<div class="form-group">
								<label for="modelName" class="col-sm-3 control-label">Model:</label>
								<div class="col-sm-9">
									<form:input path="modelName" class="form-control" required="required"/>
								</div>
							</div>
							<div class="form-group">
								<label for="generation" class="col-sm-3 control-label">Generation:</label>
								<div class="col-sm-9">
									<form:input path="generation" class="form-control"/>
								</div>
							</div>
							<div class="form-group">
								<label for="productionYear" class="col-sm-3 control-label">Production year:</label>
								<div class="col-sm-9">
									<form:input path="productionYear" class="form-control" type="number" min="1800" max="2050"  />
								</div>
							</div>
							<div class="form-group">
								<label for="doors" class="col-sm-3 control-label">Doors:</label>
								<div class="col-sm-9">
									<form:input path="doors" class="form-control" type="number" min="0" max="10"  />
								</div>
							</div>
							<div class="form-group">
								<label for="seats" class="col-sm-3 control-label">Seats:</label>
								<div class="col-sm-9">
									<form:input path="seats" class="form-control" type="number" min="0" max="50"  />
								</div>
							</div>
							<div class="form-group">
								<label for="maximumSpeed" class="col-sm-3 control-label">Max speed:</label>
								<div class="col-sm-9">
									<form:input path="maximumSpeed" class="form-control" type="number" min="0" max="550"  />
								</div>
							</div>
							<div class="form-group">
								<div class="col-sm-offset-3 col-sm-9">
									<button type="submit" class="btn btn-primary">Save</button>
								</div>
							</div>
						</form:form>
					</div>
				</div>
				</div>
			</div>
		</div>
	</body>
	</html>
	
3. ``content.jsp``:
::

	<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
		<head>
			<link href="<c:url value="${pageContext.request.contextPath}/webjars/bootstrap/3.1.0/css/bootstrap.min.css" />" rel="stylesheet">
			<link href="/resources/css/basic.css" rel="stylesheet">
	
			<script src="<c:url value="${pageContext.request.contextPath}/webjars/jquery/1.9.0/jquery.min.js"  />"></script>
			<script src="<c:url value="${pageContext.request.contextPath}/webjars/bootstrap/3.1.0/js/bootstrap.js"  />"></script>
	
			<title>CRUD - ${title}</title>
		</head>
		</tab>
			<c:import url="page_components/header.jsp"></c:import>
			<div class="container" >
				<div class="row">
					<div class="col-lg-10 col-lg-offset-1">
						<div class="panel panel-primary">
							<div class="panel-heading">
								<div class="text-center">
									<h1>${title}<small> crud operations</small></h1>
								</div>
							</div>
							<div class="alert alert-info" role="alert">
								<a class="btn btn-primary" role="button" href="/${instrument}/pdfReport?view=pdfView" target="_blank">Download PDF report</a>
								<a class="btn btn-primary" role="button" href="/${instrument}/xlsxReport.xlsx?view=excelView" target="_blank">Download Excel report</a>
							</div>
							<div class="panel-body">
								<div class="panel panel-info">
									<!-- Default panel contents -->
									<div class="panel-heading">
										<div class="text-center"><h3>Brands</h3></div>
									</div>
										<table class="table table-striped table-condensed" id="car-brands">
											<thead>
												<th><button class="sort" data-sort="brand-name">brand</button></th>
												<th><button class="sort" data-sort="founded-year">founded</button></th>
												<th><button class="sort" data-sort="headquarter">headquarter</button></th>
												<th>action</th>
											</thead>
											<tbody align="center" class="list">
												<c:forEach var="brand" items="${listCarBrand}" varStatus="status">
													<tr>
														<td class="brand-name">${brand.name}</td>
														<td class="founded-year">${brand.foundedYear}</td>
														<td class="headquarter">${brand.headquarter}</td>
														<td class="action">
															<a href="/${instrument}/edit-brand/${brand.idBrand}">Edit</a>
															&nbsp;&nbsp;&nbsp;&nbsp;
															<a href="/${instrument}/delete-brand/${brand.idBrand}">Delete</a>
														</td>
													</tr>
												</c:forEach>
											</tbody>
										</table>
									<div class="panel-footer"><a class="btn btn-info" role="button" href="/${instrument}/newBrand">Add new brand &raquo</a></div>
								</div>
	
								<div class="panel panel-info">
									<!-- Default panel contents -->
									<div class="panel-heading">
										<div class="text-center"><h3>Models</h3></div>
									</div>
										<table class="table table-striped table-condensed" id="car-models">
											<thead>
												<th><button class="sort" data-sort="brand-name">brand</button></th>
												<th><button class="sort" data-sort="model-name">model</button></th>
												<th><button class="sort" data-sort="generation">generation</button></th>
												<th><button class="sort" data-sort="production-year">produced</button></th>
												<th><button class="sort" data-sort="doors">doors</button></th>
												<th><button class="sort" data-sort="seats">seats</button></th>
												<th><button class="sort" data-sort="maximum-speed">max speed</button></th>
												<th>action</th>
											</thead>
											<tbody align="center" class="list">
												<c:forEach var="model" items="${listCarModel}" varStatus="status">
													<tr>
														<td class="brand-name">${model.carBrand.name}</td>
														<td class="model-name">${model.modelName}</td>
														<td class="generation">${model.generation}</td>
														<td class="production-year">${model.productionYear}</td>
														<td class="doors">${model.doors}</td>
														<td class="seats">${model.seats}</td>
														<td class="maximum-speed">${model.maximumSpeed}</td>
														<td>
															<a href="/${instrument}/edit-model/${model.idModel}">Edit</a>
															&nbsp;&nbsp;&nbsp;&nbsp;
															<a href="/${instrument}/delete-model/${model.idModel}">Delete</a>
														</td>
	
													</tr>
												</c:forEach>
											</tbody>
										</table>
									<div class="panel-footer"><a class="btn btn-info" role="button" href="/${instrument}/newModel">Add new model &raquo</a></div>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		<script src="/resources/js/list.min.js"></script>
		<script src="/resources/js/content-list.js"></script>
		</body>
	</html>
	
Все страницы готовы.

Добавление оформления
----------------------

Теперь добавим ресурсы CSS, JS и картинки.

Скачать необходимые ресурсы можно `отсюда <https://github.com/Xen4n/CRUD_docs/blob/master/docs/_static/resources.7z?raw=true>`_.

Создадим в директории ``webapp`` папку ``resources``. Скопируем содержимое архива в папку ``resources``.

Наш сайт готов! 
