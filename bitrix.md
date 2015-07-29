# Готовые решения по битриксу из личного опыта

### Пример работы с куками (php)
Чтение (для примера целое число)

	$visit_counter = 0;
	if ( isset($_COOKIE['visitCounter']) && is_numeric($_COOKIE['visitCounter']) ) {
		$visit_counter = $_COOKIE['visitCounter'] * 1;
	}

Запись

	setcookie("visitCounter", $visit_counter, time()+3600);  /* срок действия 1 час */

### Ajax обновление блока
    function refreshBlock(class_name) {
		var selector = '.'+class_name;
		if($(selector).length) {
			$.ajax({type: "POST", url: $(selector).attr('data-ajax'), success: function(data) {
				$(selector).html(data);				
			}});
		}
	}

### Формат даты - текущая и через 30 дней
    "DATE_ACTIVE_FROM" => ConvertTimeStamp(time(), "FULL"),
    "DATE_ACTIVE_TO" => ConvertTimeStamp(time()+ 60*60*24*30, "FULL"),

### Ресайз картинки EXACTом
    $pic = CFile::ResizeImageGet($arItem["DETAIL_PICTURE"], array('width'=>220, 'height'=>150), BX_RESIZE_IMAGE_EXACT, true);
    src="<?=$pic["src"]?>"

### Добавить цели на сайт (google)
    https://developers.google.com/analytics/devguides/collection/analyticsjs/events

### Социальные иконки на сайт
Подключаем плагин

    http://share.pluso.ru/ 

### Когда надо обрабатывать неограниченную вложенности типа каталога
можно делать (в urlrewrite)

    "CONDITION" => "#^/catalog/(.*)$#"
    
и обрабатывать в своем скрипте - так проще и логик не копипастистишь для каждой вложенности

### Примеры формирования заголовка
      $title = '';
      if($APPLICATION->getTitle()) $title = $APPLICATION->getTitle();
      if($APPLICATION->getPageProperty('title')) $title = $APPLICATION->getPageProperty('title');
      $APPLICATION->setPageProperty('title', $arResult['NAME']." - ".$arResult['SECTION_NAME']." - Проекты".$title);
      
      $APPLICATION->setPageProperty('title', $arResult['NAME']." - ".$arResult['SECTION_NAME']." -".$arResult['IBLOCK']['NAME']);

### Номер страницы в заголовок
      AddEventHandler('main', 'OnEpilog', array('CMainHandlers', 'OnEpilogHandler'));  
      class CMainHandlers { 
         public static function OnEpilogHandler() {
            if (isset($_GET['PAGEN_1']) && intval($_GET['PAGEN_1'])>0) {
               $title = $GLOBALS['APPLICATION']->GetTitle();
               $GLOBALS['APPLICATION']->SetPageProperty('title', $title.' (страница '.intval($_GET['PAGEN_1']).')');
            }
         }
      }

### Вставляем canonical ссылку при постраничке
        if (isset($_GET['PAGEN_1']) && intval($_GET['PAGEN_1'])>0) {
                $APPLICATION->AddHeadString('<link rel="canonical" href="'.$_SERVER['HTTP_HOST'].'/about/news/">');
        }

### Текущее доменное имя
      $_SERVER['HTTP_HOST']

### Проверка главной страницы
        <?if($APPLICATION->GetCurDir() == '/' && ($_SERVER['SCRIPT_NAME'] == '/index.php')):?>
                <? // тут вставляем содержимое ?>
        <?endif;?>
