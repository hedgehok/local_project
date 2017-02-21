Файл содержит примеры работы с видео при помощи утилиты ffmpeg.

Саму утилиту скачиваем отсюда: [https://ffmpeg.org/](https://ffmpeg.org/)

### Порезать видео на картинки

    ffmpeg -i video.mpg image%d.jpg
