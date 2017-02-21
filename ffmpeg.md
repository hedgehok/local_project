Файл содержит примеры работы с видео при помощи утилиты ffmpeg.

Саму утилиту скачиваем отсюда: [https://ffmpeg.org/](https://ffmpeg.org/)

Часть идей отсюда [https://habrahabr.ru/post/171213/](https://habrahabr.ru/post/171213/)

### Порезать видео на картинки

    ffmpeg -i video.mpg image%d.jpg

### Извлечь звуковую дорожку из видео и сохранить в mp3

    ffmpeg -i source_video.avi -vn -ar 44100 -ac 2 -ab 192K -f mp3 sound.mp3
