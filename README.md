# Создание простого RESTful приложения для дальнейшей реализации  JWT аутентификации на его базе

	Недавно, в процессе обучения Java, передо мной стала задача реализовать аутентификацию своего демо приложения через JSON Web Token (JWT). В сети довольно много вариантов решения данной задачи, но доступных и внятных русскоязычных ресурсов довольно мало и информация на них не всегда соответствует ожидаемому. 
	По этому я решил опубликовать эту статью основываясь на англоязычных источниках, которые мне любезно выдал Google. Быть может, она кому то пригодиться и действительно в чем то поможет. Так же приветствуються комментарии и замечания, ведь я тоже учусь и они точно не будут лишними.
	Но, что бы начать реализацию JWT, нам понадобиться какое-то не очень замысловатое приложение. Оно будет имень два метода: создание учетной записи, доступ к которому будут иметь все пользователи, и метод возвращающий приветственную строку, получить доступ к которому смогут только пользователи прошедщее аутентификацию. 	
	Назовем наше RESTful приложение testJWT и построим его на базе Spring Boot, с использованием Lombok, PostgreSQL, Liquibase и Hibernate, сделаем вид, что наше приложение - это что то действительно серьезное и создадим таблицу базы данных в ручную. Зависимости Spring Security и JWT прикрутим нашему приложению уже в процессе реализации нашей аутентификации.
	Приложение выйдет очень простое и не будет нести в себе какого тайного смысла и дополнительных проверок. Так, ввиду чисто тестового характера приложения и отсутствия критической необходимости, мы не будем проверять уникальность имен пользователей при их регистрации перед внесением в базу данных.
	Особо не терпеливые могут пропустить данную статью и сразу перейти к ее второй части (непосредственно реализации JWT аутентификции) клонировав готовое базовое приложе из моего github репозитория. Так же, пологаю что если вас интересует реализация аутентификации через JWT, осмелюсь предположить, что создание подобного простого каркасса приложения вам не в новинку. Тем не менее, рассмотрим некоторые аспекты его создания более детально.
