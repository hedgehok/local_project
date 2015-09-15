# Готовые решения по PHP и 1С-Битрикс из личного опыта

### Заготовка для компонента

[http://max22.ru/bitrix-structure/structure-component/](http://max22.ru/bitrix-structure/structure-component/)

### Создание контент-менеджера

**Задача:** 
Создать группу в которой пользователи имеют права на изменение и добавление информации в инфоблоки.

**Пример:**
Есть сайт, в нем раздел новостей (Инфоблок "Новости"), нужно создать пользователя который бы мог только вносить новости и изменять их, НО не мог менять компоненты.

**Решение:**
Создаем группу "Редактор новостей", на вкладке "ДОСТУП", вновь созданной группы, в строке "Управление структурой:" выставляем "Редактирование фалов и папок" 

Затем создаем пользователя и на вкладке "ГРУППЫ" ставим галки на: 
* Пользователи панели управления 
* Редактор новостей 

Далее нужно дать доступ на чтение нашей группе к папке

	/bitrix/admin/ 
страница

	/bitrix/admin/fileman_access.php?lang=ru&site=&path=/bitrix&files[]=admin 

Выставляем доступ на чтение для группы "Редактор новостеи" 

Теперь переходим в инфоблок "Новости" и выставляем на вкладке "ДОСТУП" для нашей группы "Редактор новостей" доступ: ИЗМЕНЕНИЕ. Вот и все, теперь авторизовавшийся на сайте пользователь сможет править новости и вносить новые. 
Краткий итог:

1. Создали группу, выставили доступ на редактирование файлов и папок

2. Создали пользователя, в внесли его в две группы:
* Пользователи панели управления 
* Редактор новостей

3. Выставили доступ на чтение для группы на на папку /bitrix/admin/

4. Дали доступ на изменение в инфоблоке

### Смена прав на файлы и папки

Найти все папки в текушей папке и вложенные и поставить на них 755

	find ./ -type d|xargs chmod 755
	
Найти все файлы в текушей папке и вложенные и поставить на них 644

	find ./ -type f|xargs chmod 644
	
Будьте внимательны, если Вы супер пользователь на сервере (личный сервер а не хостинг) вы можете случайно рекурсивно сменить
права на лишние файлы

### Простая запись логов

	file_put_contents($_SERVER['DOCUMENT_ROOT'].'/test.txt', print_r(func_get_args(), true));

### Проверка нахождения в разделе (как вариант)

	if (strpos($_SERVER['REQUEST_URI'], '/payment/') === 0) {
		...
	}

### Записывает все что передадут в /bitrix/log.txt

	function log_array() { 
	   $arArgs = func_get_args(); 
	   $sResult = ''; 
	   foreach($arArgs as $arArg) { 
	      $sResult .= "\n\n".print_r($arArg, true); 
	   } 
	
	   if(!defined('LOG_FILENAME')) { 
	      define('LOG_FILENAME', $_SERVER['DOCUMENT_ROOT'].'/bitrix/log.txt'); 
	   } 
	   AddMessage2Log($sResult, 'log_array -> '); 
	}

### Пустой компонент Bitrix - bxcert:empty

[http://marketplace.1c-bitrix.ru/solutions/beono.mastercomponent/](http://marketplace.1c-bitrix.ru/solutions/beono.mastercomponent/)

### phpinfo() из админки 1С-Битрикс

*Рабочий стол -> Настройки -> Инструменты -> Диагностика -> Настройки PHP*

### Стандартный код файла result_modifier.php

	if(!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die();
	//Подключать модуль информационных блоков не обязательно - это для наглядности
	if (CModule::IncludeModule('iblock')) {
	    //Выполнение обработки данных
	    //Тут можем работать с массивом результата $arResult
	}
	//Получение объекта текущего компонента
	$component = $this->__component;
	//Запись данных в кеш, для использования в файле component_epilog.php
	$component->setResultCacheKeys(array("KEY_ARRAY_RESULT"));

Рассмотрим простейший частный пример решения проблемы. Итак, нам необходимо получить данные из связанного элемента и добавить полученный результат в кеш. Информация по связанным элементам нам нужна в компоненте news.detail. Создаем в шаблоне компонента файл result_modifier.php и помещаем следующий код:

	if(!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die();
	//Подключим модуль информационные блоки
	if (CModule::IncludeModule('iblock')) {
	    if (isset($arResult['DISPLAY_PROPERTIES']['НАЗВАНИЕ_СВОЙСТВА'])) {
	        //Проверим чтобы точно число было
	        if (is_numeric($arResult['DISPLAY_PROPERTIES']['НАЗВАНИЕ_СВОЙСТВА']['VALUE'])
	            && $arResult['DISPLAY_PROPERTIES']['НАЗВАНИЕ_СВОЙСТВА']['VALUE']) {
	            $id = $arResult['DISPLAY_PROPERTIES']['НАЗВАНИЕ_СВОЙСТВА']['VALUE'];
	            //Вытащим все данные по элементу
	            $res = CIBlockElement::GetByID($id);
	            if ($ar_res = $res->GetNext()) {
	                if (is_numeric($ar_res['PREVIEW_PICTURE']) && $ar_res['PREVIEW_PICTURE']) {
	                    //Тут например можем обработать превью изображение
	                }
	                $arResult['НОВЫЙ_КЛЮЧ_В_МАССИВЕ_РЕЗУЛЬТАТА'] = $ar_res;
	            }
	        }
	    }
	}
	
	$component = $this->__component;
	//Добавим в кеш полученный результат
	$component->setResultCacheKeys(array('НОВЫЙ_КЛЮЧ_В_МАССИВЕ_РЕЗУЛЬТАТА'));
	
Теперь в файле template.php и component_epilog.php будет доступна информация по связанному элементу, чтобы получить ее надо обратиться к массиву результата:

	echo $arResult['НОВЫЙ_КЛЮЧ_В_МАССИВЕ_РЕЗУЛЬТАТА']['NAME'];

### Создание фильтра для компонента

	global $arrFilterName;
	$arrFilterName = array("PROPERTY_NAME_CODE" => true);

При вызове компонента

	....
	"FILTER_NAME" => "arrFilterName",
	....

### Восстановление админ доступа через FTP

[http://bx-cert.ru/advices/34/uteryan-parol-ot-admina-no-est-ftp/](http://bx-cert.ru/advices/34/uteryan-parol-ot-admina-no-est-ftp/)

### Как уменьшить занимаемое место системой 1С-Битрикс

[http://bx-cert.ru/advices/34/kak-umenshit-zanimaemoe-mesto-sistemoy-bitriks/](http://bx-cert.ru/advices/34/kak-umenshit-zanimaemoe-mesto-sistemoy-bitriks/)

### Устанавливаем чистую версию 1С-Битрикс

[http://bx-cert.ru/advices/34/kak-ustanovit-chistuyu-cms-1s-bitriks/](http://bx-cert.ru/advices/34/kak-ustanovit-chistuyu-cms-1s-bitriks/)

### Защита от множественных ajax запросов
Имеем:
1. Форму фильтра с кучей контроллов
2. Обработчики на каждый контролл, суть которых сводится к вызову функции обновления рабочей области (подгрузка отфильтрованных данных через ajax)
3. Функцию ajax загрузки и обновления рабочей области по success

Защита от частых кликов по контроллам формы решается прерыванием ajax запроса:

	var cfAjax;
	function catalogFilterAjax(){
		...
		if(cfAjax && cfAjax.readyState != 4){
			cfAjax.abort();
		}
		cfAjax = $.ajax({
			...
		});
	}

### Local Redirect в битриксе

	LocalRedirect('/tarifs/');
	LocalRedirect('/tarifs/', false, '301 Moved permanently');

[http://dev.1c-bitrix.ru/api_help/main/functions/other/localredirect.php](http://dev.1c-bitrix.ru/api_help/main/functions/other/localredirect.php)

Через htaccess: 

	RewriteRule ^tarifs/index.php$ /tarifs/ [R=301,L]

### Получить элемент по коду

	function GetElementByCode($IBLOCK, $ELEMENT_CODE) {
		if(CModule::IncludeModule("iblock")) { 
			$arSelect = Array("IBLOCK_ID", "ID", "NAME", "PREVIEW_TEXT", "PREVIEW_PICTURE", "DETAIL_TEXT", "DETAIL_PICTURE", "PROPERTY_*", "DETAIL_PAGE_URL");
			$arFilter = Array("IBLOCK_ID"=>$IBLOCK, "ACTIVE"=>"Y", "CODE"=>$ELEMENT_CODE);
			$res = CIBlockElement::GetList(
				array("SORT"=>"ASC"), 
				$arFilter, 
				false, 
				array(), 
				$arSelect
			);
			$item = $res->GetNext();
			return $item;
		} 
	}

### Страница поиска без стандартного битрикс компонента - вариант фильтра

Для примера поиск по названию и символьному коду

	$q = split(' ', trim(htmlspecialchars($_REQUEST['q'])));
	$filter = array("LOGIC" => "AND");
	foreach ($q as $val) {
		$filter[] = array(	
						"LOGIC" => "OR",
						'%NAME' => trim($val),
						'%CODE' => trim($val)
					);
	}
	$GLOBALS['arrFilter'][] = $filter;

Далее arrFilter суем в какой надо селект компонент

### В чем различие в результате массива ключей $arResult[NAME] от $arResult[~NAME]?

С тильдой - это НЕбезопасные данные. Грубо говоря, с тильдой - это результат от Fetch, а без тильды - GetNext. Грубо говоря.

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
