# Облачные платформы как основа технологического предпринимательства

**Университет:** Университет ИТМО  
**Дисциплина:** Облачные платформы как основа технологического предпринимательства  
**Студент:** Кухи Арина (Arina Kuhi)  
**Группа:** U4225  
**Учебный год:** 2026

## Лабораторные работы

| № | Тема | Отчёт | Статус |
|---|---|---|---|
| 1 | Обзор Google Cloud и исследование основных сервисов | [Лабораторная №1](lab1/lab1_report.md) | Выполнена |
| 2 | Исследование Cloud Run | [Лабораторная №2](lab2/lab2_report.md) | Выполнена |
| 3 | Исследование Cloud Storage | [Лабораторная №3](lab3/lab3_report.md) | Выполнена |

---

# Лабораторная работа №1. Обзор Google Cloud и исследование основных сервисов

**Дата выполнения:** 03.11.2026  
**Цель:** познакомиться с Google Cloud, IAM, Compute Engine и Cloud Storage, исследовать права доступа.

[Полный отчёт по лабораторной №1](lab1/lab1_report.md)

## 1. Создание сервисного аккаунта

В проекте `cloud-platforms-as-the-basis` создан сервисный аккаунт `akuhi-sa-lab1@cloud-platforms-as-the-basis.iam.gserviceaccount.com`. Первоначально назначена роль **Storage Admin**.

![Создание сервисного аккаунта](lab1/images/01-service-account.png)

![Роль Storage Admin](lab1/images/02-storage-admin.png)

## 2. Создание виртуальной машины

| Параметр | Значение |
|---|---|
| Имя | `akuhi-vm-lab1` |
| Регион / зона | `us-central1` / `us-central1-a` |
| Тип машины | `e2-micro` |
| Модель предоставления | Spot |
| ОС | Debian GNU/Linux 13 |
| Диск | 10 GB |
| Сервисный аккаунт | `akuhi-sa-lab1` |

![Конфигурация Spot VM](lab1/images/03-spot-e2-micro.png)

![ОС и диск](lab1/images/04-debian-disk.png)

## 3. Подключение по SSH и копирование файлов

Подключение к VM выполнено через браузерный SSH-терминал. Команды:

```bash
gcloud auth list
gcloud storage ls gs://lab1-bucket-itmo/
mkdir -p ~/lab1-files
gcloud storage cp gs://lab1-bucket-itmo/* ~/lab1-files/
ls -lah ~/lab1-files/
```

**Результат:** успешно загружены три файла: `pic1.jpg` (228 KB), `pic2.jpg` (487 KB), `pic3.jpeg` (858 KB). В терминале отобразилось `Completed files 3/3`.

![Подключение SSH](lab1/images/05-ssh.png)

## 4. Проверка IAM

Роль аккаунта заменена с **Storage Admin** на **Compute Viewer**. После этого повторное скачивание файла успешно выполнилось:

```bash
gcloud storage cp gs://lab1-bucket-itmo/pic2.jpg ~/lab1-test/pic2-new.jpg
```

Выяснилось, что бакет `lab1-bucket-itmo` дополнительно предоставлял группе `allUsers` роль **Storage Object Viewer**, что объясняет успешное чтение объектов после смены роли сервисного аккаунта.

![Роль Compute Viewer](lab1/images/06-compute-viewer.png)

![Разрешения бакета](lab1/images/07-bucket-permissions.png)

![Разрешения allUsers](lab1/images/08-allusers.png)

## 5. Завершение и вывод

Виртуальная машина `akuhi-vm-lab1` удалена. Исследованы базовые сервисы GCP и показано, что итоговый доступ складывается из всех действующих разрешений IAM, в том числе назначенных на уровне бакета.

---

# Лабораторная работа №2. Исследование Cloud Run

**Дата выполнения:** 03.11.2026  
**Цель:** исследовать развёртывание контейнеров, мониторинг, ревизии и распределение трафика в Cloud Run.

[Полный отчёт по лабораторной №2](lab2/lab2_report.md)

## 1. Создание сервиса

Создан Cloud Run сервис `akuhi-lab2` в регионе `us-central1` с публичным доступом и контейнером:

```text
us-docker.pkg.dev/cloudrun/container/hello
```

Изначально использовались порт `8080`, минимум `0` экземпляров и максимум `1` экземпляр.

![Форма создания сервиса](lab2/images/01-create-service.png)

![Выбор контейнерного образа](lab2/images/02-container-image.png)

## 2. Запуск и наблюдаемость

Открытие URL сервиса показало страницу **It's running!**. Создана первая ревизия `akuhi-lab2-00001-rx4`.

![Первая ревизия](lab2/images/03-hello-first-revision.png)

В интерфейсе Observability изучены **Logs** и **Metrics**. На момент проверки ненулевых значений на графиках не было; страница Hello обновлялась для генерации HTTP-запросов.

![Metrics](lab2/images/04-metrics.png)

## 3. Изменение порта

В настройках контейнера порт изменён с `8080` на `8090`. После повторного развёртывания создана ревизия `akuhi-lab2-00002-fvs`. Страница **It's running!** продолжила открываться.

![Изменение порта](lab2/images/05-port-8090.png)

![Вторая ревизия](lab2/images/06-hello-second-revision.png)

## 4. Управление трафиком

Первоначально новая ревизия получала 100% трафика, а старая — 0%.

![История ревизий](lab2/images/07-revisions-before.png)

При попытке распределить трафик поровну получена ошибка:

```text
service level max instances must be greater than or equal to the number of targets receiving traffic.
```

После увеличения ограничения максимального числа экземпляров сервиса до `2` успешно применено распределение:

| Ревизия | Порт | Трафик |
|---|---|---|
| `akuhi-lab2-00001-rx4` | 8080 | 50% |
| `akuhi-lab2-00002-fvs` | 8090 | 50% |

![Распределение трафика 50/50](lab2/images/08-traffic-50-50.png)

## 5. Завершение и вывод

Сервис `akuhi-lab2` удалён. Изучены развёртывание Cloud Run, конфигурация контейнера и перераспределение запросов между ревизиями. Проверено влияние ограничений масштабирования на настройку трафика.

---

# Лабораторная работа №3. Исследование Cloud Storage

**Дата выполнения:** 03.11.2026  
**Цель:** научиться создавать бакеты, загружать объекты, организовывать их по префиксам и управлять публичным доступом через IAM.

[Полный отчёт по лабораторной №3](lab3/lab3_report.md)

## 1. Создание бакета

В проекте `cloud-platforms-as-the-basis` создан бакет со следующими характеристиками:

| Параметр | Значение |
|---|---|
| Название | `arina-kuhi-lab3-2026` |
| Регион | `us-central1` (Iowa) |
| Класс хранения | Standard |
| Модель контроля доступа | Uniform |
| Публичный доступ на момент создания | Not public |
| Защита | Soft Delete |

![Выбор расположения бакета](lab3/images/01-bucket-location.png)

![Созданный бакет](lab3/images/02-bucket-created.png)

## 2. Загрузка и организация объектов

В корень бакета через **Upload → Upload files** загружены три PNG-изображения.

![Три изображения в корне бакета](lab3/images/03-files-in-root.png)

Создана папка-префикс `image/`. Файлы из корня были удалены и повторно загружены в неё. Это была именно **повторная загрузка**, а не операция Move.

```text
arina-kuhi-lab3-2026/
└── image/
    ├── Снимок экрана — 2026-11-03 в 20.40.24.png
    ├── Снимок экрана — 2026-11-03 в 20.40.27.png
    └── Снимок экрана — 2026-11-03 в 20.40.31.png
```

![Три изображения в папке](lab3/images/04-files-in-folder.png)

## 3. Настройка разрешений

Сначала бакет был непубличным.

![Изначальные права](lab3/images/05-permissions-before.png)

Через **Permissions → Grant access** группе `allUsers` назначена роль **Storage Object Viewer**. В результате интерфейс показал `Access granted to public principals`.

![Публичный доступ](lab3/images/06-public-access.png)

Разрешение действует на объекты всего бакета при Uniform bucket-level access, а не на одну папку.

## 4. Проверка публичных ссылок

Ссылки на PNG-файлы были открыты в браузере через `storage.googleapis.com`. Изображения отображались без авторизации.

![Публичное изображение 1](lab3/images/07-public-image-1.png)

![Публичное изображение 2](lab3/images/08-public-image-2.png)

Пример структуры ссылки (не действующий URL после удаления бакета):

```text
https://storage.googleapis.com/arina-kuhi-lab3-2026/image/<имя-файла>
```

## 5. Завершение и вывод

После проверки публичных ссылок учебный бакет `arina-kuhi-lab3-2026` удалён. Освоены создание Cloud Storage, загрузка файлов, логическая организация объектов и настройка доступа IAM. Продемонстрирован эффект публичной роли `Storage Object Viewer` для `allUsers`.

---

## Итоги

| Лабораторная | Сервисы и технологии | Результат |
|---|---|---|
| №1 | Compute Engine, IAM, Cloud Storage | VM, SSH, копирование трёх файлов, анализ ролей |
| №2 | Cloud Run, Revision History, Metrics | Две ревизии, изменение порта, трафик 50/50 |
| №3 | Cloud Storage, IAM | Бакет, три изображения, публичный доступ, проверка ссылок |

## Источники

- [Google Cloud Documentation](https://cloud.google.com/docs)
- [Google Cloud Storage](https://cloud.google.com/storage/docs)
- [Google Cloud Run](https://cloud.google.com/run/docs)
- [Задание ИТМО — лабораторная №1](https://ex-itmo-ict-faculty.github.io/cloud-platforms-as-the-basis-of-technology-entrepreneurship/education/labs2023-2024/lab1/lab1/)
- [Задание ИТМО — лабораторная №2](https://ex-itmo-ict-faculty.github.io/cloud-platforms-as-the-basis-of-technology-entrepreneurship/education/labs2023-2024/lab2/lab2/)
- [Задание ИТМО — лабораторная №3](https://ex-itmo-ict-faculty.github.io/cloud-platforms-as-the-basis-of-technology-entrepreneurship/education/labs2023-2024/lab3/lab3/)
