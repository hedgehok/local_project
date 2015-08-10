# Готовые решения по PHP и 1С-Битрикс из личного опыта

### Почему надо юзать GetNext а не GetNextElement

GetNextElement просто обертка [http://bxapi.ru/src/?module_id=iblock&name=CIBlockResult::GetNextElement](http://bxapi.ru/src/?module_id=iblock&name=CIBlockResult::GetNextElement) Если простыми словами - вернет и поля, и свойства (хотя само является объектом). Обратитесь к документации. Не рекомендуется юзать без четкого понимания.
	
GetNext - простая выборка, возвращает то, что попросили.

Fetch - то же самое, но возвращает данные в небезопасном виде (не применено htmlspecialchars). Рекомендуется использовать с пониманием что это действительно надо (когда случится, поймете). 

### Битрис - вывод только для админа
Удобно успользовать для вывода отладочной инфы:

	global $USER;
	if ($USER->IsAdmin()) { echo "Что то принтим"; }
	
Также доступны:

	if ($USER->IsAuthorized()) { // ... }
	if (C$USER->IsOnLine()) { // ... }

### Простой и прикольный прогресс бар

[http://ricostacruz.com/nprogress/](http://ricostacruz.com/nprogress/)

Пример использования: [sorax.org](http://sorax.org/)

### Scroll to top

	$('.js-jump-top').on('click', function(e) {
		e.preventDefault();
		$('html, body').animate({'scrollTop': 0});
	});

### Заморозить скролл страницы находясь на диве

[http://jsbin.com/itajok](http://jsbin.com/itajok)

### Простая кастомизация скроллбара

	<link href="../css/jquery.scrollbar.css" rel="stylesheet">
	<script src="../js/jquery.scrollbar.min.js"></script>
	
	<div class="scrollbar-inner"> <!-- some block content --> </div>
	
	<script type="text/javascript">
		$('.scrollbar-inner').scrollbar();
	</script>

[https://github.com/gromo/jquery.scrollbar/](https://github.com/gromo/jquery.scrollbar/)

### Facebook обнуление кешированной share-инфы

	https://developers.facebook.com/tools/debug/

### Кнопка шаринга Faceboook своими руками

	var href = window.location.href,
	    $fb_counter = $main_social_list.find('.social-item-counter.fb > span'),
	    $fb_link = $main_social_list.find('.social-item-link.fb');
	    
	// fb
	$fb_link.on('click', function(e){
		e.preventDefault();
		window.open(
			    'https://www.facebook.com/sharer/sharer.php?src=sp&u=' + encodeURIComponent(location.href)+
			    //'&t=' + encodeURIComponent('title')+
			    //'&description=' + encodeURIComponent('description')+
			    //'&picture=' + encodeURIComponent(document.location.origin + image) +
			    //'&noparse=1'+
			    '',
			    'fb-share-dialog',
			    'width=626,height=436'
			);
	});
	$.getJSON('http://graph.facebook.com/?id=' + encodeURI(href) + '&callback=?', function(response) {
		if (response.shares !== undefined) {
			$fb_counter.html(response.shares);
		}
	});

### Кнопка шаринга Вконтакте своими руками
По сути кнопка шаринга состоит из двух частей - ссылка для всплывающего окна шары и счетчика количества поделившихся. Реализуем примерно так:

	// vk
	var href = window.location.href,
	    $vk_link = $main_social_list.find('.social-item-link.vk');
	    
	$vk_link.on('click', function(e){
		e.preventDefault();
		window.open(
			    'http://vk.com/share.php?url=' + encodeURIComponent(location.href)+
			    //'&title=' + encodeURIComponent('title')+
			    //'&description=' + encodeURIComponent('description')+
			    //'&image=' + encodeURIComponent(document.location.origin + "/img/soc_img.jpg") +
			    //'&noparse=1' +
			    '',
			    'vk-share-dialog',
			    'width=626,height=436'
			);
	});
	var VK = {
		Share: {
			count: function(value, count) {
			    //$('#vkontakte_count').html(count);
			    $('#main-social-list').find('.social-item-counter.vk > span').html(count);
			}
		}
	}
	$.getJSON('http://vkontakte.ru/share.php?act=count&index=1&url=' + encodeURI(href) + '&callback=?', function(response) {});
	
объект VK нужен для авторазбора ответа от ВКшного сервера.
Ответ поступает в виде:

	VK.Share.count(1, 75);
	
Где 75 - количество поделившихся.

### Свойства страницы для шаринга в соцсети
Добавляем метатеги:

	<meta property="og:image" content="http://allsoft.ru/bitrix/templates/allsoft2011/images/8let/dragon_normal.jpg" />
	<meta property="og:title" content="Я – лицензионный Дракон!" />
	<meta property="og:description" content="Результат теста: Дракон почти Ваш «конек»! Вы пока не можете преподавать Драконоведение, но на верном пути!" />
	
Тут подробнее: [http://habrahabr.ru/company/softline/blog/144946/](http://habrahabr.ru/company/softline/blog/144946/)

### Простой класс реализующий редирект
	class Utils {
	    public static function redirect($uri = '') {
	        header("HTTP/1.1 301 Moved Permanently");
	        header("Location: ".$uri, TRUE, 302);
	        exit;
	    }
	}

применение:

	Utils::redirect('ya.ru');

### Вытаскиваем элемент по коду (без компонента)
$_REQUEST["ELEMENT_CODE"] настраиваем в urlrewrite

	if (isset($_REQUEST["ELEMENT_CODE"])) {
		CModule::IncludeModule('iblock');
		$res = CIBlockElement::GetList(
					Array(), 
					Array("IBLOCK_ID"=>10, CODE=>$_REQUEST["ELEMENT_CODE"]), 
					false, 
					Array(), 
					Array("ID", "NAME", "CODE", "PREVIEW_TEXT", "DETAIL_TEXT")
				);
		if($ob = $res->GetNextElement()) {
			$arFields = $ob->GetFields();
			$APPLICATION->AddChainItem($arFields["NAME"]);
			$APPLICATION->SetTitle($arFields["NAME"]);
			?>
				<header class="title-light">
					<h1><?=$arFields["NAME"]?></h1>
				</header>
				<div class="text-block-ekibastuz font-16"> 
					<p><strong><?=$arFields["PREVIEW_TEXT"]?></strong></p>
					<?=$arFields["DETAIL_TEXT"]?>
				</div>
			<?
		} else {
			CHTTP::SetStatus("404 Not Found");
			@define("ERROR_404","Y");
		}
	}

### Пример работы с сессией
	session_start();
	$_SESSION['pages'][] = $_SERVER['PHP_SELF'];
	
	if ( isset($_SESSION['name']) ) {
		echo $_SESSION['name'];
	}

### Пример работы с куками (php)
Чтение (для примера целое число)

	$visit_counter = 0;
	if ( isset($_COOKIE['visitCounter']) && is_numeric($_COOKIE['visitCounter']) ) {
		$visit_counter = $_COOKIE['visitCounter'] * 1;
	}
	
	$last_visit = '';
	if ( isset($_COOKIE['lastVisit']) ) {
		$last_visit = htmlspecialchars($_COOKIE['lastVisit'], ENT_QUOTES);
		$last_visit = stripslashes(trim($last_visit));
	}

Запись

	setcookie("visitCounter", $value);
	// или со сроком действия
	setcookie("visitCounter", $visit_counter, time()+3600);  /* срок действия 1 час */
	setcookie("lastVisit", date('d/m/Y H:i:s'));

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
