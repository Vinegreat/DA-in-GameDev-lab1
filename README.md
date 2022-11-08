# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
# 3. Лабораторная работа. Разработка системы машинного обучения

Отчет по лабораторной работе #3 выполнил(а):
- Черемухина Екатерина Вячеславовна
- НМТ-211507
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.

## Цель работы
Познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.


## Постановка задачи.
В данной лабораторной работе мы создадим ML-агент и будем тренировать нейросеть, задача которой будет заключаться в управлении шаром. Задача шара заключается в том, чтобы оставаясь на плоскости находить кубик, смещающийся в заданном случайном диапазоне координат.



## Задание 1. 
###  Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity. При выполнении задания можно использовать видео-материалы и исходные данные, предоставленные преподавателями курса.
-	Создим новый пустой 3D проект на Unity.

-	Скачайем папку с ML агентом. 

![image](https://user-images.githubusercontent.com/114614965/197861114-91a9725b-4965-4487-97cf-bf2c5230c945.png)

-	В созданный проект добавляем ML Agent, выбрав Window - Package Manager - Add Package from disk. Последовательно добавляем .json – файлы:
	ml-agents-release_19 / com,unity.ml-agents / package.json
	ml-agents-release_19 / com,unity.ml-agents.extensions / package.json

![image](https://user-images.githubusercontent.com/90757310/197793620-d9c7d792-8f99-4355-80a6-a113d683d5dd.png)

-	Далее запускаем Anaconda Prompt для возможности запуска команд через консоль.

![image](https://user-images.githubusercontent.com/90757310/197793868-91705f59-0d51-4245-949a-d475069483e5.png)

-	Далее пишем серию команд для создания и активации нового ML-агента, а также для скачивания необходимых библиотек:
-	
	`mlagents 0.28.0`

![image](https://user-images.githubusercontent.com/90757310/197794052-44ac3201-b68b-403e-bfd7-98bcb5d13877.png)

	torch 1.7.1

![image](https://user-images.githubusercontent.com/90757310/197794150-ee542e25-fbd2-41c4-80b1-a1f1d182101c.png)

-	Создадим на сцене плоскость, куб и сферу, а затем простой C# скрипт и подключис его к сфере:

![image](https://user-images.githubusercontent.com/114614965/197861302-dc49c5b9-3d04-44b3-b839-ed1172f0bf20.png)

-	В скрипт-файл RollerAgent.cs добавляем код:


```C#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }

}


```

-	Объекту «сфера» добавим компоненты Rigidbody, Decision Requester, Behavior Parameters и настройте их.

![image](https://user-images.githubusercontent.com/90757310/197795056-c8536eec-ef3a-429d-bcd7-3e1e582f543e.png)

-	В корень проекта добавим файл конфигурации нейронной сети.

![image](https://user-images.githubusercontent.com/90757310/197795253-f351ba8e-647c-48fd-af6e-1a3ab957d8f6.png)

- Запустим ml-агента

![image](https://user-images.githubusercontent.com/90757310/197795363-e4b8ad40-a327-4217-bed0-caad79b6cb70.png)


-	Сделаем 3, 9, 27 копий модели «Плоскость-Сфера-Куб», запустим симуляцию сцены.

![image](https://user-images.githubusercontent.com/114614965/197860943-9ca05ed1-1cab-4d38-a591-002ee2c67ad5.png)

-	После завершения обучения проверьте работу модели.



https://user-images.githubusercontent.com/114614965/197861964-e285379d-5897-41dd-a194-2a355715b422.mp4



### Выводы

 PyTorch —  является ядром вычислений при обучении нашего ML-агента. После создания ML Агента он может действовать с заранее заданными настройками, указанными в конфиге его тренера. Настройки тренера серьезно влияют на скорость и качество обучения Агента. Количество управляемых объектов прямопропорционально его обучаемости. Одновременно может обучаться несколько разных агентов при коррекции конфига их тренера.

 ## Задание 2. 
### Подробно опишите каждую строку файла конфигурации нейронной сети, доступного в папке с файлами проекта по ссылке. Самостоятельно найдите информацию о компонентах Decision Requester, Behavior Parameters, добавленных на сфере.

В файле `rollerball config.yaml` содержится конфигурация тренера YAML, в которой находятся все значения гиперпарамеров. Фактически содержание этого конфига регламентирует обучаемость агента.


`behaviors`

 Основной раздел файла конфигурации тренера — это набор конфигураций для каждого поведения в сцене. Они определены в подразделе behaviors файла конфигурации тренера.
 Параметры существующие в среде и описанные в нашем конфиге behaviors будут иметь значения, которые мы им присвоили непосредственно в конфиге,
 а не описанные параметры примут значения по умолчанию. 
  
  
`RollerBall`
	Вернемся к моменту обучения нейронной сети, когда MLAgent имеет статус active и пакет “mlagents==0.28.0” и фреймворк “torch~1.7.1” были загружены, и проанализируем введенную команду, сравнив со стандартным обращением к агенту:
  
`mlagents-learn rollerball_config.yaml --run-id=RollerBall --resume`

`mlagents-learn <trainer-config-file> --env=<env_name> --run-id=<run-identifier>`

Нас интересует обращение к конфигу `rollerball_config.yaml` и id `RollerBall`
где `<run-identifier>` - это это уникальное имя, которое используется для идентификации результатов тренировок нейронной сети. Поскольку внутри одной среды может быть несколько объектов под контролем нейронной сети, они могут обучаться одновременно, но различным вещам. Например в то время, как RollerBall стремится к прикосновению с Target, Target может прятаться или убегать от RollerBall. В таком случае нам не придется учить их отдельно, мы добавим соответствующий раздел в конфигурацию обучения под id Target и внесем изменения в скрипт внутри Unity, где опишем поощрение результата “Сбежал” для нашего Target-кубика.


`trainer_type: ppo`
Сам параметр отвечает за тип используемого тренера (по умолчанию ppo). Помимо ‘ppo’ существуют такие типы тренера как ‘sac’ и ‘poca’.
Proximal Policy Optimization [PPO] - класс алгоритмов обучения с подкреплением, которые работают сравнимо или даже лучше, чем современные подходы, но при этом их гораздо проще реализовать и настроить. Для нас важно то, что это совокупность алгоритмов обучения, которой нам удобнее пользоваться для решения большинства задач. 


`hyperparameters`
	Это раздел настройки гиперпараметров для агента. Перейдем непосредственно к параметрам, которые мы затронули в нашем конфиге 

`batch_size`
	Количество опытов в каждой итерации градиентного спуска. Это всегда должно быть в несколько раз меньше, чем количество опытов для обновления модели политики(изменения поведенческой модели). Если мы используем непрерывные действия, это значение должно быть большим (порядка 1000 с). Но мы используем только дискретные действия, поэтому значение должно быть меньше (порядка 10 с).

`buffer_size`
	Количество опытов, которые необходимо собрать перед обновлением модели политики. Соответствует тому, сколько опыта должно быть собрано, прежде чем мы будем изучать или обновлять модель. Значение должно быть в несколько раз больше, чем batch_size . Обычно большее buffer_size значение соответствует более стабильным обновлениям обучения.

`learning_rate`
	Начальная скорость обучения для градиентного спуска. Соответствует силе каждого шага обновления градиентного спуска. Обычно это значение следует уменьшать, если обучение нестабильно, а вознаграждение не увеличивается постоянно.

`beta`
	Энтропия - мера хаоса в машинном обучении.
beta - сила регуляризации энтропии, которая делает политику «более случайной». Это гарантирует, что агенты должным образом исследуют пространство действия во время обучения. Увеличение этого параметра обеспечит выполнение большего количества случайных действий. Необходимо так скорректировать параметр, чтобы энтропия (измеряемая с помощью TensorBoard) медленно уменьшалась вместе с увеличением вознаграждения. Если энтропия падает слишком быстро, увеличиваем beta. Если энтропия падает слишком медленно, уменьшаем beta.


`epsilon`
	Влияет на скорость изменения политики во время обучения. Соответствует допустимому порогу расхождения между старой и новой политикой при обновлении градиентного спуска. Установка небольшого значения этого параметра приведет к более стабильным обновлениям, но также замедлит процесс обучения.

`lambd`
	Параметр регуляризации (лямбда), используемый при расчете обобщенной оценки преимущества ( GAE ). Лямбду можно рассматривать как то, насколько агент полагается на свою текущую оценку стоимости при вычислении обновленной оценки стоимости. Низкие значения соответствуют большему полаганию на текущую оценку ценности (что может быть высоким смещением), а высокие значения соответствуют большему полаганию на фактические вознаграждения, полученные в среде (что может быть высокой дисперсией). Параметр обеспечивает компромисс между ними, и правильное значение может привести к более стабильному процессу обучения.


`num_epoch`
	Количество проходов через буфер опыта при выполнении оптимизации градиентного спуска. Чем больше размер партии, тем больше это допустимо. Уменьшение этого параметра обеспечит более стабильные обновления за счет более медленного обучения.


`learning_rate_schedule`
	(по умолчанию = linear для PPO и constant для SAC) Определяет, как скорость обучения изменяется с течением времени. Для PPO в документации рекомендовано снижать скорость обучения до значения max_steps, чтобы обучение сходилось более стабильно. Однако в некоторых случаях (например, обучение в течение неизвест	ного времени) эту функцию можно отключить. Для SAC мы рекомендуем поддерживать скорость обучения постоянной, чтобы агент мог продолжать обучение до тех пор, пока его функция Q не сойдется естественным образом.


`linear` 
	скорость обучения уменьшается линейно, достигая 0 на max_steps, сохраняя при constant этом скорость обучения постоянной для всего тренировочного прогона.


`network_settings`
	Раздел, содержащий настройки нейронной сети.


`normalize`
	Параметр определяет применяется ли нормализация к входным данным векторных наблюдений. Эта нормализация основана на скользящем среднем и дисперсии векторного наблюдения. Нормализация может быть полезна в случаях со сложными задачами непрерывного управления, но может быть вредна для более простых задач дискретного управления. По умолчанию false.


`hidden_units`
	Количество единиц в скрытых слоях нейронной сети. Соответствуют количеству единиц в каждом полносвязном слое нейронной сети. Для простых задач, где правильное действие представляет собой простую комбинацию входных данных наблюдения, это значение должно быть небольшим. Для задач, где действие представляет собой очень сложное взаимодействие между переменными наблюдения, это значение должно быть больше.


`num_layers`
	Количество скрытых слоев в нейронной сети. Соответствует количеству скрытых слоев после ввода наблюдения или после кодирования CNN визуального наблюдения. Для простых задач меньше слоев, скорее всего, будут обучать быстрее и эффективнее. Для более сложных задач управления может потребоваться больше слоев.


`reward_signals`
	Раздел позволяет задавать настройки как для внешних (т. е. основанных на среде), так и для внутренних сигналов вознаграждения (например, любопытство и GAIL). Каждый сигнал вознаграждения должен определять как минимум два параметра: strength и gamma, в дополнение к любым гиперпараметрам, специфичным для класса. Обратите внимание: чтобы удалить сигнал вознаграждения, вы должны полностью удалить его запись из файла reward_signals. По крайней мере, один сигнал вознаграждения должен оставаться определенным в любое время.


`extrinsic`
	Dнешние сигналы вознаграждения(основанные на среде)
 

`gamma`
	(по умолчанию = 0.99) Фактор скидки для будущих вознаграждений, поступающих из окружающей среды. Это можно рассматривать как то, как далеко в будущем агент должен заботиться о возможных вознаграждениях. В ситуациях, когда агент должен действовать в настоящем, чтобы подготовиться к вознаграждению в отдаленном будущем, это значение должно быть большим. В случаях, когда вознаграждение является более немедленным, оно может быть меньше. Должен быть строго меньше 1.


`strength`
(по умолчанию = 1.0) Коэффициент, на который умножается вознаграждение, данное средой. Типичные диапазоны будут варьироваться в зависимости от сигнала вознаграждения.


`max_steps`
Общее количество шагов (т. е. собранных наблюдений и предпринятых действий), которые необходимо выполнить в среде (или во всех средах при параллельном использовании нескольких) перед завершением процесса обучения. Если в вашей среде есть несколько агентов с одинаковым именем поведения, все шаги, предпринятые этими агентами, будут max_steps учитываться в одном и том же счетчике.



`time_horizon`
Сколько шагов опыта необходимо собрать для каждого агента, прежде чем добавить его в буфер опыта. Когда этот предел достигается до конца эпизода, оценка значения используется для прогнозирования общего ожидаемого вознаграждения из текущего состояния агента. Таким образом, этот параметр является компромиссом между менее предвзятой, но более высокой оценкой дисперсии (длинный временной горизонт) и более предвзятой, но менее разнообразной оценкой (короткий временной горизонт). В тех случаях, когда в эпизоде ​​есть частые награды или эпизоды непомерно велики, более идеальным может быть меньшее количество. Это число должно быть достаточно большим, чтобы охватить все важные действия в последовательности действий агента.


`summary_freq`
(по умолчанию = 50000) Количество опытов, которое необходимо собрать перед созданием и отображением статистики обучения. Это определяет детализацию графиков в Tensorboard.


### Компоненты	

- `Decision Requester`


Цель обучения с подкреплением — изучить наилучшую политику (сопоставление состояний с действиями), которая максимизирует возможные вознаграждения. 
Во время обучения агент будет либо выполнять действия:

Наугад - любопытство (чтобы узнать, какие действия приводят к вознаграждению, а какие нет)
Из его текущей политики (оптимальное действие при текущем состоянии)

Компонент “Decision Requester” выбирает, какое действие будет принято в данном эпизоде обучения.

`Decision Period`

Соответсвенно отвечает за частоту опроса решений из базы знаний агента. Значение 10 означает, что решение будет запрашиваться раз в 10 шагов агента.

`Take Actions Between Decision Period`

Отвечает за то, будет ли агент принимать решения во время отработки действий из базы знаний.  Т.е. Агент может изменять корректировать их, если сочтет необходимым.


- `Behavior Parameters`

Агент — это действующее лицо, которое наблюдает и действует в окружающей среде. В среде RollerBall базовый объект RollerAgent имеет несколько свойств, влияющих на его поведение:
Параметры поведения — у каждого агента должно быть поведение. Поведение определяет, как агент принимает решения.

Максимальный шаг — определяет, сколько шагов моделирования может произойти до окончания эпизода агента. В RollerBall агент не перезапускается, поэтому мы обнуляем этот параметр.

Параметры поведения: векторное пространство наблюдения

Прежде чем принять решение, агент собирает свои наблюдения о своем состоянии в мире. Вектор наблюдения представляет собой вектор чисел с плавающей запятой, который содержит информацию, необходимую агенту для принятия решений. Т.е. они прямо отсылаюстся на “Decision Requester”

`Space Size`
Длина вектора состояния "мозга"(в непрерывном пространстве состояний) или количество возможных значений(в дискретном пространстве)
Агенту даются инструкции в виде действий. ML-Agent классифицирует действия на два типа: непрерывные и дискретные. RollerBall запрограммирован на использование дискретного действия, касания цели, которое изменит статус в момент касания, и непрерывного действия - перемещения, а также отслеживания координат цели или целей. Так в первом задании у нас этот параметр был равен 8 и в нем содержалось 3 (xyz) цели, 3 (xyz) сферы под управлением агента и 2 (x-y) вектора для перемещения сферы. А в третьем задании добавились координаты второй цели, параметр изменил минимальное значение на 11. При несоответствии параметра с реальным числом активных "наблюдений" происходит исключиение вызова эвристического метода и прямая отсылка к несоответствию Space Size

`Stacked Vectors`
Количество состояний, которые будут склаываться перед подачей в нейронную сеть.

`Actions`

`Continuous Actions`
Количество непрерывных действий.

`Discrete Branches`
Кол-во дискретных ветвей - наборов событий для получения награды.

`Model` 
Обученная модель поведения, вкладывается после успешного обучения агента. Характеризует модель поведения объекта под управлением нейронной сети.

`Interface Device`
Устройство для генерации поведения объекта под управлением нейронной сети.

`Deterministic Interface` 
Детерминированный выход означает, что определенный набор входных данных всегда будет генерировать одну и ту же модель поведения.

`Behavior Type`
Тип поведения. Отвечает за функционал агента. Например, невозможно обучить агента, если данное поле имеет параметр только вывод, поскольку он подразумевает использование накопленной информации. Также не эффективен параметр только ввод, если мы хотим пользоваться готовой моделью ИИ. 

`Team ID`
Параметр для выбора команды для агента или ИИ. Например, если мы делаем спортивный симулятор, то мы будем обучать агентов одному и тому же, но играть они будут против друг друга.

`Use Child Sensors`
Использует все компоненты Sensor, прикрепленные к дочерним GameObjects для этого агента. Мы использовали sensors для определения местоположения тела под управлением агента в пространстве и местонахождения цели, а также отслеживания velocity - вращения сферы для перемещения агента. 

`Observable Attribute Hand`
Изучение агентом всевозможных атрибутов. Может исключаться, изучить все или только дочерние атрибут для тела под управлением агента.



## Задание 3. 
###  Доработайте сцену и обучите ML-Agent таким образом, чтобы шар перемещался между двумя кубами разного цвета. Кубы должны, как и в первом задании, случайно изменять координаты на плоскости.

По условию нам необходимо, чтобы шар перемещался между кубами разного цвета. Это можно реализовать, отслеживая координаты каждого куба и различая их по тегу или имени. 
Я выбрал вариант различия по имени, а также добавил несколько флажков в скрипт RollerAgent для создания неболшой логики.

```C#

void Start()
    {
        rBody = GetComponent<Rigidbody>();
        ColorLastTarget = ColorGreen;
    }
    public Transform BlueTarget;
    public Transform GreenTarget;
    public string ColorBlue = "Blue"; 
    public string ColorGreen = "Green";
    public string ColorLastTarget;
    ...
    float distanceToGreen = Vector3.Distance(this.transform.localPosition, GreenTarget.localPosition);
    float distanceToBlue = Vector3.Distance(this.transform.localPosition, BlueTarget.localPosition);

        if(distanceToBlue < 1.42f && ColorLastTarget == ColorGreen)
        {
            SetReward(1.0f);
            EndEpisode();
            ColorLastTarget = ColorBlue;
        }
        else if(distanceToGreen < 1.42f && ColorLastTarget == ColorBlue)
        {
            SetReward(1.0f);
            EndEpisode();
            ColorLastTarget = ColorGreen;
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }


```
После дополнения скрипта и пространства Unity получилось следующее

![Норм_Trim](https://user-images.githubusercontent.com/90757310/197833063-2855a3b6-9e91-42d9-bcea-e90a4c3f299e.gif)


## Выводы

В процессе выполнения 3 лабораторной работы я обучила ML-агента для реализации ИИ в среде Unity, чтобы это реализовать потребовалось воспользоваться библиотеками mlagents 0.28.0 (torch версии 1.7.1 PyTorch отвечает за вычислительные функции в процессе). Для выполнения второго задания я ознакомилась с документацией по созданию тренера агента, для третьего - научила агента различать объекты по цвету, который придает материал, поэтому я выбрала подход инкапсуляции тега GameObject/имени GameObject. 

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
