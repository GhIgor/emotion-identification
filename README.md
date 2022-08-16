# Итоговый проект курса «Data Scientist. ML. Средний уровень (нейронные сети)»
Данная работа включает в себя создание классификационной модели распознавания эмоций. 
Коротко о техническом задании, а именно, какой должна быть модель:
1)	На вход изображение – на выходе наиболее вероятная эмоция из представленных (anger, contempt, disgust, fear, happy, neutral, sad, surprise, uncertain). После создания модели, ее необходимо проверить – проклассифицировать 5000 изображений. Создать csv-файл в котором две колонки: 1) Имя файла с расширением; 2) Тип эмоции. 
Решения будут оцениваться по метрике: «categorical_accuracy». 
Критерии оценки: 
«Зачет»:  0.29880 ≤ public leaderboard;
«4»: 0.29880 ≤ public leaderboard и private leaderboard;
«5»: 0.4 ≤ public leaderboard и private leaderboard.
2)	Время инференса сети на Google Colab не должно превышать 0,33 секунды (3 кадра в секунду).
3)	Приветствуется использование архитектур свёрточных нейронных сетей, разобранных во время теоретических занятий.
4)	«Улучшение и дополнения к заданию». Обернуть модель в Python-класс с методами: 
a. __init__, в котором будет происходить загрузка весов модели.
b. predict, который будет принимать на вход предобработанный кадр из веб-камеры и возвращать эмоцию.
Исходные данные: 50 046 изображений, сгруппированных по девяти папкам, каждая папка соответствует названию эмоции.
![image](https://user-images.githubusercontent.com/85408423/184788301-725ca87b-ab3e-4e0c-baaa-cad12a102f7e.png)
Мы видим, что количество изображений в группах эмоций – не сбалансировано. Например, изображений в папке «anger» (7 022) больше, чем сумма изображений по группам «contempt» (3 085) и «disgust» (3 155).
##Этапы выполнения работы:

### **Анализ данных:** 0_Data_analysis.ipynb. 
В процессе анализа данных было выявлено несколько проблем: 
1) *Есть изображения на которых лицо не определяется.*
Данные изображения были исключены.
2) *Есть изображения на которых присутствует несколько лиц.*
Все изображения были обрезаны по bounding box и оставлял box, который был больше по площади. Пользовался детектором лиц MTCNN – он лучше справлялся с задачей поиска лица на изображении, чем детектор в OpenCV (в работе приведен пример).
3) *Дубликаты изображений. Одинаковые изображения могут находится в разных папках (группах эмоций).*
Нашел способ определения одинаковых изображений не просто по хешу (файлы одинаковые по весу). Определил одинаковые изображения, даже если они имели разный размер или смещение лица. Главным условием было, чтоб лицо полностью попало в кадр. Если все дубликаты находились в одной папке – оставлял только одно изображение, если дубликаты найдены в разных папках – удалял все (считал разметку данных ошибочной).

### **Выбор модели:** 1_MobileNetV2.ipynb, 2_VGG19.ipynb, 3_ResNet50.ipynb.
В правилах указано: «Работа требует реализации алгоритма, с помощью которого будет задана заданная классификация. Разрешается использовать предварительно обученные сети только на датасете ImageNet. Приветствуется использование архитектурных сверточных нейросетей, разобранных во время теоретических занятий.»
Во время теоретических занятий были рассмотрены следующие архитектуры нейросетей, предварительно обученных на датасете ImageNet:
1)	MobileNetV2;
2)	VGG19;
3)	ResNet50.
Опыты проводились в равных условиях. Один набор входных данных с одинаковыми параметрами аугментации: поворот изображения до 20 градусов, случайное отражение по горизонтали.  Соотношение тренировочных/валидационных данных = 0.8/0.2.
К базовой архитектуре добавлялось два слоя: GlobalAveragePooling2D и полносвязный слой.
Обучались модели на 15 эпохах двумя путями:
1)	Все веса базовых архитектур заморожены, обучались только добавленные слои.
2)	Все веса разморожены.
Модель выбиралась по двум параметрам: скорость инференса и максимальная метрика «categorical_accuracy» на валидационных данных.

![image](https://user-images.githubusercontent.com/85408423/184789100-24460f26-7f60-433e-88d4-c5cdee57c276.png)

Во всех случаях, модель давала лучший прогноз при обучении всех весов модели. В итоге для дальнейшего обучения, была выбрана архитектура модели: VGG19 – по сравнению с другими моделями она дала больше точность и быстрый инференс.

### **Продолжение обучение модели:** 4_VGG19_cont.ipynb.
В данном блоке оставил код загрузки входных данных в модель, аугментации и результат обучения модели на 15 эпохах.
Последующее обучение на 5 эпохах не принесло положительного результата, т.к. модель переобучалась (точность на тестовой выборке росла, а на валидационной – снижалась).
Снизил скорость обучения в 10 раз и точность на первой эпохе незначительно увеличилась до 0.53055.
Далее заморозил все слои, кроме последнего и обучал модель еще 7 эпох, в результате точность на валидационной выборке составила 0.533.
Получилось увеличить точность только на 0.6%.
Ускорить модель, оптимизировав ее с помощью библиотеки TensorRT не получилось, ранее это делал, используя более позднюю версию tensorflow 1.х, но сейчас такой путь не работает.

### **Проверка модели на тестовых данных:** 5_Test.ipynb
В качестве проверки точности распознавания эмоций предоставлен файл test_kaggle.zip. Он содержит 5000 изображений. В данном файле написан код, в котором модель представлена в виде класса. Далее загрузка тестовых данных и формирование csv-файла для отправки на платформу Kaggle. В нём для каждого изображения с лицом человека должна указываться наиболее вероятная эмоция из представленных.
Применяя внутри класса детектор лиц MTCNN, удалось достичь точности: Public Score – 0.50560 и Private Score – 0.50400. Но время инференса вместе с обработкой изображения занимает 750 мс, из которых 487 мс – занимает детектор MTCNN. 
Время инференса сети на Google Colab не должно превышать 0,33 секунды (3 кадра в секунду). В связи с этим, внутри класса был использован детектор из OpenCV, его время работы составляет 31.2 мс. В результате удалось уменьшить время инференса до 297 мс, но точность немного понизилась и составляет: Public Score – 0.48640 и Private Score – 0.48920.

### **Итоговый результат и применение модели:** 6_vebcam.ipynb.
Оставил блок с разбором как использовать веб-камеру. 
Написал класс для создание модели, которая определяет эмоции на изображении, даже если на нем присутствует группа людей.
В заключении представлены 2 класса модели для идентификации эмоции в режиме реального времени через веб-камеру: 1) с детектором MTCNN; 2) с детектором из OpenCV – более быстрая модель.


