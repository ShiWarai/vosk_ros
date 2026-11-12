# vosk_ros
Пакет ROS 2 для запуска STT ноды на основе Vosk

## 📋 Описание

Этот пакет предоставляет узел ROS 2 для непрерывного распознавания речи с использованием библиотеки Vosk. Он адаптирован из проекта [whisper_ros](https://github.com/mgonzs13/whisper_ros) для работы с Vosk и без использования GPU.

### Основные отличия от whisper_ros:
- **Движок STT**: Использует [Vosk](https://alphacephei.com/vosk/) вместо Whisper.cpp.
- **Без GPU**: Разработан для работы на CPU, не требует CUDA или ONNX Runtime с GPU.
- **Аудиовход**: Подписывается на ROS2 топик `/audio/input` (публикуется `audio_capturer_node` из пакета `audio_common`).
- **VAD**: Работает непрерывно, без Voice Activity Detection (VAD).
- **Совместимость интерфейсов**: Сохраняет совместимость с сообщениями и действиями `whisper_msgs` (`Transcription.msg`, `STT.action`) для легкой интеграции.
- **Модель**: Модель Vosk не включена в репозиторий. Её необходимо скачать отдельно.

## Зависимости

- Python пакеты:
  - `vosk` - библиотека распознавания речи
  - `numpy` - работа с массивами данных
  - `rclpy` - ROS2 Python клиент

- ROS2 пакеты:
  - `std_msgs` - стандартные сообщения ROS2
  - `rosidl_default_generators` - генерация интерфейсов
  - `audio_common_msgs` - сообщения для аудио
  - `audio_common` - пакет для захвата и воспроизведения аудио через PortAudio ([https://github.com/mgonzs13/audio_common](https://github.com/mgonzs13/audio_common))

## Установка

```bash
# Перейдите в директорию src вашего рабочего пространства ROS2
cd ~/ros2_ws/src

# Клонируйте этот репозиторий
git clone <repository_url>

# Перейдите в корневую директорию рабочего пространства
cd ~/ros2_ws

# Установите зависимости (если еще не установлены)
rosdep install --from-paths src --ignore-src -r -y

# Клонируйте audio_common (если еще не клонирован)
cd ~/ros2_ws/src
git clone https://github.com/mgonzs13/audio_common.git
cd ~/ros2_ws

# Установите Python зависимости
pip3 install vosk numpy

# Соберите пакет
colcon build --packages-up-to vosk_ros

# Загрузите окружение ROS2
source install/setup.bash
```

## Скачивание модели Vosk

Модель Vosk не включена в репозиторий из-за большого размера. Вам необходимо скачать её вручную.

1. **Перейдите на сайт Vosk:**
   [https://alphacephei.com/vosk/models](https://alphacephei.com/vosk/models)

2. **Скачайте нужную модель (например, `vosk-model-small-ru-0.22.zip`):**
   ```bash
   wget https://alphacephei.com/vosk/models/vosk-model-small-ru-0.22.zip
   ```

3. **Распакуйте модель в директорию `models` внутри пакета `vosk_ros`:**
   ```bash
   unzip vosk-model-small-ru-0.22.zip -d ~/ros2_ws/src/vosk_ros/models/
   # Убедитесь, что путь к модели внутри пакета выглядит так:
   # ~/ros2_ws/src/vosk_ros/models/vosk-model-small-ru
   ```
   Или укажите путь к модели через параметр `model_path` при запуске узла.

## Использование

### Запуск узла

**Важно**: Перед запуском `vosk_node` необходимо запустить источник аудио данных (например, `audio_capturer_node` или Telegram бот):

```bash
# Терминал 1: Запустите audio_capturer_node (для микрофона)
ros2 launch rds_2p_audio audio_nodes.launch.py

# Терминал 2: Запустите vosk_node
ros2 launch vosk_ros vosk.launch.py
```

### Запуск для Telegram аудио

Для работы с голосовыми сообщениями из Telegram используйте специальный launch файл:

```bash
ros2 launch vosk_ros vosk_telegram.launch.py
```

Этот launch файл автоматически настроен для работы с Telegram:
- `audio_topic` = `/telegram/audio`
- `transcription_topic` = `/telegram/audio/transcription` (формируется автоматически)
- `default_sample_rate` = `16000` Hz (для ранней инициализации)
- `default_channels` = `1` (моно)

### Параметры запуска

**Основные параметры:**
- `model_path` - путь к директории модели Vosk (по умолчанию используется модель из пакета)
- `audio_topic` - ROS2 топик для подписки на аудио данные (по умолчанию `/audio/input`)
<<<<<<< Updated upstream
- `transcription_topic` - имя топика для публикации результатов распознавания (по умолчанию `/audio/transcription`)
- `max_audio_time` - максимальное время накопления аудио в секундах перед принудительным сбросом, защита от переполнения (по умолчанию `30.0`)
=======
- `transcription_topic` - имя топика для публикации результатов распознавания (по умолчанию формируется автоматически как `/audio_topic/transcription`)
- `max_audio_time` - максимальное время накопления аудио в секундах перед принудительным сбросом (по умолчанию `30.0`)
- `silence_timeout` - таймаут тишины в секундах для публикации результата после окончания потока данных (для файлов, по умолчанию `2.0`)
>>>>>>> Stashed changes

**Параметры ранней инициализации:**
- `default_sample_rate` - частота дискретизации по умолчанию для ранней инициализации recognizer (по умолчанию `44100`, установите `0` для ленивой инициализации)
- `default_channels` - количество каналов по умолчанию (по умолчанию `1`)

**Примечание:** Если `transcription_topic` не указан явно или равен дефолтному значению `/audio/transcription`, он автоматически формируется из `audio_topic` по принципу `/название_топика/transcription`. Например:
- `audio_topic` = `/telegram/audio` → `transcription_topic` = `/telegram/audio/transcription`
- `audio_topic` = `/audio/input` → `transcription_topic` = `/audio/input/transcription`

### Примеры запуска

**Стандартный запуск (микрофон):**
```bash
ros2 launch vosk_ros vosk.launch.py
```

**Для Telegram аудио:**
```bash
ros2 launch vosk_ros vosk_telegram.launch.py
```

**С кастомными параметрами:**
```bash
ros2 launch vosk_ros vosk.launch.py \
    audio_topic:="/my_audio_topic" \
    transcription_topic:="/my_transcription_topic" \
    default_sample_rate:=16000 \
    default_channels:=1 \
    silence_timeout:=2.0
```

**С ранней инициализацией (для известного формата):**
```bash
ros2 launch vosk_ros vosk.launch.py \
    audio_topic:="/audio/input" \
    default_sample_rate:=44100 \
    default_channels:=1
```

## Топики

**Подписка:**
- `/audio/input` (`audio_common_msgs/msg/AudioStamped`) - подписка на аудио данные (по умолчанию, можно изменить через параметр `audio_topic`)
- Другие топики аудио настраиваются через параметр `audio_topic`

**Публикация:**
- `/audio/transcription` (`rds_2p_audio/msg/Transcription`) - публикует результаты распознавания речи (по умолчанию, формируется автоматически из `audio_topic`)
- Топик транскрипции формируется автоматически как `/audio_topic/transcription`, если не указан явно

**Примеры:**
- `audio_topic` = `/audio/input` → `transcription_topic` = `/audio/input/transcription`
- `audio_topic` = `/telegram/audio` → `transcription_topic` = `/telegram/audio/transcription`

## Действия (Actions)

- `/vosk/listen` (`rds_2p_audio/action/STT`) - действие для запроса распознавания речи

## Сообщения

Интерфейсы общие для `vosk_ros` и `gigaam_ros` и находятся в пакете `rds_2p_audio`.

### Transcription
- `text` (string) - распознанный текст
- `audio_time` (float32) - время аудио
- `transcription_time` (float32) - время распознавания

## 📝 Лицензия

MIT License