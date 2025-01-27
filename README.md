# LayerZero script
Практически AIO скрипт для LayerZero.  
Скрипт написан закрытым сообществом **"Спекуляция"**, подписывайтесь - https://t.me/spekulyantcrypto  
Кодеры, которые писали скрипт:  
1) **Крипто-Шелки** - https://t.me/tawer_crypt  
2) **Кузница Гефеста** - https://t.me/gefest_forge
------------------------------------------------
***Скрипт подготовлен исключительно для ознакомительных целей.***   
***This script was developed for educational purposes only.***  
  
***Код открытый. Не доверяете - проверяйте весь код или не пользуйтесь. Приватные ключи никуда не отправляются, дрейнеры не цепляются. Если ваши средства уйдут куда-то не туда, вины скрипта в этом нет.*** 
------------------------------------------------
Если скрипт вам пригодился, будем рады вашему донату в любой EVM-сети: 0x9d9D67FAF623a2D78A1eaa07579b7128fCD79dd9
------------------------------------------------
## Что умеет скрипт:
- Бриджить ВСЕ(!!!) имеющиеся USDT/USDC из любой сети в любую сеть (кроме ETH, FTM (из-за деактивации пулов), Metis)
- Покупать STG в POLYGON и стейкать их
- Покупать BTC в AVALANCHE, бриджить их через btcbridge в POLYGON, бриджить обратно и продавать
- Свапать ETH в ARBITRUM или OPTIMISM на GoerliETH через https://testnetbridge.com/, в одну сторону
- Бриджить USDT в Aptos через https://theaptosbridge.com/bridge, в одну сторону
- Бриджить USDT в Harmony через https://bridge.harmony.one/erc20, в одну сторону
- Разбавлять активности транзакции свапами в разных сетях
- Работать в мультипотоке
- Выбирать рандомные кошельки и выполнять активности в рандомном порядке
- Записывать общие логи всех действий и отдельно по кошелькам

# Установка и запуск
Устанавливаем библиотеки на ПК или venv (лучше и безопаснее):  
**ПК** - `pip install -r requirements.txt`  
**venv** - `python -m venv venv`, потом `venv\Scripts\activate`, и `pip install -r requirements.txt`  

Запуск скрипта происходит через файл `execute.py`.  
Файл запускаем через любой IDE (ex. VS Code), либо через командную строку.  

Для запуска через Cmd: Win+R, cmd, enter, пишем `cd путь_к_папке`, пишем `python execute.py`.
# Подготовка к использованию
Для удобства работы с *.csv файлами, желательно скачать Rons CSV Editor, либо любую другую программу для работы с csv, под эксель не подстраивали. Ссылка на Rons https://www.ronsplace.ca/Products/RonsEditor/Download.  
Сети в скрипте делятся на single и не-single. Single сети - ARBITRUM, BSC, OPTIMISM - сети, ИЗ (from) которых можно бриджить, а В (to) - нельзя. Сделано для экономии, т.к. бридж в эти сети в ~2 раза дороже, чем из этих сетей.
Не-single сети - POLYGON, AVALANCHE - основные сети.

**Если у вас нет опыта в программировании, перечитайте инструкцию - проведите первоначальную настройку, а другой код не меняйте.**  
**Также настоятельно рекомендуем сначала запустить прогон 2-3х кошельков, для понимания логики работы скрипта.**  
**Скрипт работает в связке с Refuel - https://github.com/speculyaciya/Refuel_bungee, но именно нашим скриптом пользоваться необязательно, просто желательно иметь в каждой сети запас нативной валюты, чтобы скрипт не выбрасывал ошибку недостаточного баланса, сам он не рефулит.**

### main_config.py
`WAIT_TIME = range(5, 10)` - задержка между действиями, в минутах  
`RATIO_STARGATE_SINGLE = 0.5`  - процент от баланса USDT/USDC, который будет использован для бриджа из single сетей (1 = 100%, 0.5 = 50%)  
`STARGATE_CHAIN_LIST` - список не-single сетей. Можно добавлять свои (которые представлены на Stargate, естественно). Если нет желания копаться в коде - скип, т.к. для описания процесса добавления пришлось бы писать отдельный гайд.

### max_setting.csv
Таблица с настройками комиссий (в долларах).  
`Activity` - активность  
`MAX_GAS` - максимальная комиссия за транзакцию (газ)  
`MAX_VALUE` - максимальный value Stargate (см. скрин ниже)  

![](https://skr.sh/i/260623/WdX1jNvH.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82%2026-06-2023%2021:39:30.jpg)

### RPCs.py
Заполняете своими или паблик рпц.  

Бесплатные приватные рпц:
1. https://www.alchemy.com/ (с ними не было проблем)
2. https://www.quicknode.com/
3. https://accounts.lavanet.xyz/ (иногда отваливались)

### stargate_settings.py
*Опционально!*  
Здесь можно изменить маршруты бриджей. 

Для примера возьмем ARBITRUM:  
В `chain_to` указаны POLYGON и AVALANCHE. Если мы хотим добавить возможность бриджить из ARBITRUM в OPTIMISM - добавляем `'OPTIMISM',` с новой строки.

### data.csv
**Сердце скрипта! Уделите особое внимание заполнению этой таблицы.**
*Во всех десятичных числах (17.89) пишется точка ("."), а не запятая.*

1. `DO` - нужно ли прогонять этот кошелек. Чтобы скрипт начал его прогонять, нужно написать английскую "X". Без "X" не важно, какие активности выбраны дальше. После окончания прогона всех выбранных активностей кошелька, "X" поменяется на "DONE".
2. `Name` - название кошелька (необходимо для логов).
3. `Wallet` - адрес кошелька в EVM-сети.
4. `Private_key` - приватный ключ от кошелька.
5. `Proxy` - поле для прокси. Пока не работает, оставляем пустым.
6. `Stargate` - лог - общее количество бриджей через старгейт после запуска скрипта, изменять ничего не нужно, нужно для отчётности вам. Изначально = 0.
7. `Stargate_range` - сумма от и до - какое количество купить USDT/USDC в не-single сетях. Если на ваших кошельках нет USDT/USDC в не-single сетях, то скрипт приобретет рандомную сумму из заданного ренжа в рандомной сети (!!!).
8. `STARGATE_FIRST_SWAP` - указать "DONE" если приобретать USDT/USDC в не-single сетях не нужно (подразумевается, что на одной из сетей уже есть стейблы для прогона). Если покупка стейблов необходима - оставляем пустым.
9. `STARGATE_POLYGON/AVALANCHE` - какое количество бриджей из этой сети нужно сделать через StarGate. Изначально 0. **ВАЖНО!** Должен стоять 0, а не пустая строка, иначе выдаст ошибку.
11. `STARGATE_BSC` - по аналогии с `STARGATE_POLYGON/AVALANCHE`, только для BSC.
9. `STARGATE_BSC_RANGE` - значение, сколько покупать USDT/USDC в BSC, по аналогии с `STARGATE_RANGE`.
10. `STARGATE_BSC_FIRST_SWAP` - по аналогии с `STARGATE_FIRST_SWAP`, только для BSC.
11. `STARGATE_ARBITRUM` - по аналогии с `STARGATE_POLYGON/AVALANCHE`, только для ARBITRUM.
11. `STARGATE_ARBITRUM_RANGE` - по аналогии с `STARGATE_POLYGON/AVALANCHE`, только для ARBITRUM.
12. `STARGATE_ARBITRUM_FIRST_SWAP` - по аналогии с `STARGATE_FIRST_SWAP`, только для ARBITRUM.
13. `STARGATE_LIQ` - добавление ликвидности, пока не работает.
14. `STARGATE_LIQ_VALUE` - сколько ликвидности добавить на Stargate, пока не работает.
15. `STARGATE_STG` - нужно ли покупать и стейкать STG. Модуль работает на POLYGON.
16. `STARGATE_STG_RANGE` - сумма от и до в $, ренж сколько покупать STG для стейка.
17. `BTC_BRIDGE` - сколько бриджей делать из AVALANCHE в POLYGON и обратно с покупкой-продажей BTC.b. **ВАЖНО!** Должен стоять 0, а не пустая строка, иначе выдаст ошибку.
18. `BTC_BRIDGE_RANGE` - в каком диапазоне покупать BTC.b в $ эквиваленте для работы.
19. `BTC_BRIDGE_STEP` - этапы `BTC_BRIDGE`, трогать не нужно. Изначально пусто. С каждым этапом будет появляться "X", обозначающий, на каком этапе находится скрипт:
Один X - купил BTC.b. Два X  - забриджил BTC.b в аваланч из полигона. Три X  - забриджил BTC.b в полигон из аваланча. Пустая строка - скрипт выполнил продажу BTC.b и вычел из колонки `BTC_BRIDGE` 1 (единицу).
20. `TESTNET_BRIDGE` - сколько раз свапать ETH в GETH через testnetbridge.
21. `TESTNET_BRIDGE_RANGE` - диапазон покупки GETH в $ эквиваленте.
21. `TESTNET_BRIDGE_CHAINS` - какие сети использовать в testnetbridge. Выбирает случайную. Писать через запятую, с большой буквы.
22. `APTOS_BRIDGE` - сколько раз бриджить USDT из BSC в APTOS.
23. `APTOS_BRIDGE_RANGE` -  в каком диапазоне покупать USDT/USDC.
24. `APTOS_BRIDGE_WALLET` - Уникальный Aptos кошелек, на который будут бриджиться USDT. (Генерируйте на [Cointool](https://cointool.app/createWallet/aptos "Cointool") для удобства)
25. `HARMONY_BRIDGE` - сколько раз бриджить USDT из BSC в HARMONY.
26. `HARMONY_BRIDGE_RANGE` - в каком диапазоне покупать USDT/USDC для бриджа.
27. `SWAP_TOKEN` - рандомные свапы, чтобы запутать алгоритм активностей для выявления паттернов сбрива. *(Опционально)*
28. `SWAP_TOKEN_RANGE` - в каком диапазоне свапать рандомные токены. *(Опционально)*
29. `SWAP_TOKEN_CHAINS` - в каких сетях делать случайные свапы. *(Опционально)*

# Логи
В папке `log_wallet` будут появляться текстовые файлы с логами по конкретным кошелькам. Это необходимо для удобства отслеживания возможных багов и ошибок.  
В файле `log.csv` ведется весь лог по активностям. Если в конце строки написано "True" - значит активность завершилась успешно. Если же написано "False" - значит активность завершилась с ошибкой. Также в ячейке с "False" будет указана встреченная ошибка.  
В файл `main.txt` дублируется лог из Cmd / IDE. В `log_wallet` то же самое, но с разбивкой по кошелькам.

# Примечания
В случае, если во время работы скрипта произошла ошибка, скрипт будет двигаться дальше, пока не завершит все активности и не останутся те активности, которые ему не удается выполнить (не хватает средств, проблема с пулом и т.д.). Все ошибки можно посмотреть в логах. 
Касательно всех действий в сети Оптимизм, с ним аккуратно, т.к. все рпцшки оптимизма лагают, даже если приватные, 1инч постоянно выкидывает bad request, хоть и потом всё отрабатывает, это никак не починить.

**Постарайтесь не останавливать скрипт во время выполнения. Если же все-таки без остановки не обойтись - дождитесь выполнения активности и появления строки `Запуск в...`, а после, остановите скрипт комбинацией клавиш Ctrl+Pause Break. Иначе вам придется разбираться, на каком этапе вы прервали скрипт и редактировать `data.csv`**
