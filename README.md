# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил(а):
- Куц Евгений Александрович
- НМТ-213901
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | # | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.
Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Цель работы
Ознакомиться с программными средствами для организции передачи данных между инструментами google, Python и Unity.

## Задание 1
### Реализовать совместную работу и передачу данных в связке Python - Google-Sheets – Unity.
В начале выполнения работы я создал проект в Google Cloud, где подключил API для работы с Google Sheets и создал сервисный аккаунт, через который данные отправляются в гугл-таблицу.

!картиночка

После настройки передачи данных в таблицу, был написан код на Python, который генерировал случайные числа (подразумевавшие цены), а также рассчитывал инфляцию по этим ценам и загружал их в таблицу (код представлен ниже)

```py

import gspread
import numpy as np
gc = gspread.service_account(filename = 'unitydatascience-364716-a19ce910ca41.json')
sh = gc.open("UnitySheets")
price = np.random.randint(2000, 10000, 11)
mon = list(range(1,11))
i = 0
while i <= len(mon):
i += 1
if 1 == 0:
continue
else:
tempInf = ((price[i-1]-price[i-2])/price[i-2])*100
tempInf = str(tempInf)
tempInf = tempInf.replace('.',',')
sh.sheet1.update(('A' + str(i)), str(i))
sh.sheet1.update(('B' + str(i)), str(price[i-1]))
sh.sheet1.update(('C' + str(i)), str(tempInf))
print(tempInf)

```

Скриншоты результатов в PyCharm и Google таблицах прилагаются:

!картиночка
!картиночка

Далее в Юнити был создан 3D проект, куда на сцену были добавлены пустой объект и два пакета с библиотеками и звуками. После этого я написала скрипт, получающий из гугл-таблицы значения и анализирующий их, после чего по сценарию воспроизводящий звук в зависимости от значения инфляции и выводящий значение инфляции (код представлен ниже).

```py

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
public AudioClip goodSpeak;
public AudioClip NormalSpeak;
public AudioClip BadSpeak;
private AudioSource selectAudio;
private Dictionary<string,float> dataSet = new Dictionary<string, float>();
private bool statusStart = false;
private int i = 1;

// Start is called before the first frame update
void Start()
{
StartCoroutine(GoogleSheets());
}

// Update is called once per frame
void Update()
{

}

IEnumerator GoogleSheets()
{
UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1Hh4k1yLxHaXOYZ91YbDJ8okh_ogDMRsV_kM2R5THEug/values/Лист1?key=AIzaSyArFzRfcD7kaCJ0uwtkx8mPgANlECuY2e0");
yield return curentResp.SendWebRequest();
string rawResp = curentResp.downloadHandler.text;
var rawJson = JSON.Parse(rawResp);
foreach (var itemRawJson in rawJson["values"])
{
var parseJson = JSON.Parse(itemRawJson.ToString());
var selectRow = parseJson[0].AsStringList;
dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
}
Debug.Log(dataSet["Mon_1"]);
}
}

```
Пустой объект получил набор звуков и к нему был прикреплен написанный скрипт:

!картиночка

Результат запуска сцены:

!картиночка

## Выводы

В результате выполнения данной работы я научился работать с программными средствами для организции передачи данных между инструментами google, Python и Unity, и создавать работу в связке этих инструментов.

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
