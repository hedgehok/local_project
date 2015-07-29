# Готовые решения по битриксу из личного опыта

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
