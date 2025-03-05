<?
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/header.php"); 
$APPLICATION->SetTitle("test");
$APPLICATION->SetAdditionalCSS("/test/style.css");
?>
<?
global $USER;
$userId = $USER->GetID();
if ($USER->IsAdmin()):?>
<?php
if(!CModule::IncludeModule("iblock"))
return;

$iblock_id = 30;
// $arrSection = array();
// $arFilter = array('IBLOCK_ID' => $iblock_id);
// $rsSections = CIBlockSection::GetList(array('NAME' => 'ASC'), $arFilter);
// $index = 0;
// while ($arSection = $rsSections->Fetch())
// {
// 	$arrSection[$index]["NAME"] = $arSection["NAME"];
// 	$arrSection[$index]["ID"] = $arSection["ID"];
// 	$index++;
// }


$arPlacemarks = array();
$arrKompany = array();
$indexK = 0;
$arSelectK = Array("ID", "IBLOCK_ID", "NAME", "PROPERTY_*");

if($_POST["name"]){
	$arFilterK = Array("IBLOCK_ID"=>$iblock_id, "NAME"=>"%".$_POST["name"]."%", "ACTIVE_DATE"=>"Y", "ACTIVE"=>"Y");
} elseif($_GET["id"]){
	$arFilterK = Array("IBLOCK_ID"=>$iblock_id, "ACTIVE_DATE"=>"Y", "ACTIVE"=>"Y", "ID"=>$_GET["id"]);
} else {
	$arFilterK = Array("IBLOCK_ID"=>$iblock_id, "ACTIVE_DATE"=>"Y", "ACTIVE"=>"Y");
}

$resK = CIBlockElement::GetList(Array('NAME' => 'ASC'), $arFilterK, false, Array(), $arSelectK);
while($obK = $resK->GetNextElement()){ 
	$fields = $obK->GetFields();  
	$prop = $obK->GetProperties();

	$arrKompany[$indexK]["ID"] = $fields["ID"];
	$arrKompany[$indexK]["NAME"] = $fields["NAME"];
	$arrKompany[$indexK]["ATT_WEB"] = $prop["ATT_WEB"]["VALUE"];
	$arrKompany[$indexK]["ATT_PHONE"] = $prop["ATT_PHONE"]["VALUE"];
	$arrKompany[$indexK]["ATT_EMAIL"] = $prop["ATT_EMAIL"]["VALUE"];
	$arrKompany[$indexK]["ATT_ADDRESS"] = $prop["ATT_ADDRESS"]["VALUE"];

	$res = CIBlockElement::GetByID($prop["ATT_OFFICE_CENTER"]["VALUE"]);
	if($ar_res = $res->GetNext()){
		$arrKompany[$indexK]["LOGO"] = CFile::GetPath($ar_res["PREVIEW_PICTURE"]); 
	}

	if($_GET["id"] && $fields["ID"] == $_GET["id"]){
		$onMap = explode(",", $prop['ATT_MAP']['VALUE']);
		$mapLAT = $onMap[0];
		$mapLON = $onMap[1];
		$arPlacemarks[] = array(
			"LAT" => $mapLAT,
			"LON" => $mapLON,
			"TEXT" => '<img style="width: 100px;" src="'.$arrKompany[$indexK]["LOGO"].'"><br><b>'.$fields["NAME"].'</b><br>Адрес: '.$prop["ATT_ADDRESS"]["VALUE"].'<br>почта: '.$prop["ATT_EMAIL"]["VALUE"].'<br>тел.:'.$prop["ATT_PHONE"]["VALUE"][0].'<br>сайт: <a target="_blank" href="mailto:'.$prop["ATT_WEB"]["VALUE"].'">'.$prop["ATT_WEB"]["VALUE"].'</a>',
		);
	} elseif(!$_GET["id"]) {
		$onMap = explode(",", $prop['ATT_MAP']['VALUE']);
		$mapLAT = $onMap[0];
		$mapLON = $onMap[1];
		$arPlacemarks[] = array(
			"LAT" => $mapLAT,
			"LON" => $mapLON,
			"TEXT" => '<img style="width: 100px;" src="'.$arrKompany[$indexK]["LOGO"].'"><br><b>'.$fields["NAME"].'</b><br>Адрес: '.$prop["ATT_ADDRESS"]["VALUE"].'<br>почта: '.$prop["ATT_EMAIL"]["VALUE"].'<br>тел.:'.$prop["ATT_PHONE"]["VALUE"][0].'<br>сайт: <a target="_blank" href="mailto:'.$prop["ATT_WEB"]["VALUE"].'">'.$prop["ATT_WEB"]["VALUE"].'</a>',
		);
	}
	$indexK++;
}
?>

<div class="block_map">
	<div class="block_map_abs">
		<div>
			<?if($_GET["id"]):?>
				<a href="/test/" class="block-border block-border-hover textUp wid-100">Показать всех дистрибьютеров</a>
			<?else:?>
				<form id="search_distrib" name="search_distrib" action="/test/" method="POST" enctype="multipart/form-data">
					<input class="inp90 textNum" type="text" name="name" maxlength="name" value="" placeholder="Введите название">
					<button class="btn_distr" type="submit" form="search_distrib" value="Submit" title="Искать">Искать</button>
				</form>
				<div class="block-border textUp">Дистрибьюторы</div>
			<?endif;?>
		</div>
		<?if(empty($arrKompany)){?>
			<br>
			<center>
			<h4>Ничего не найдено</h4>
			</center>
			<br>
		<?} else {?>
		<div class="scroll-block">
			<?foreach($arrKompany as $company):?>
				<div class="pad_inl_10 bg_hover" id="<?=$company['ID']?>">
					<div class="dis_fl_sb">
						<?if($company["LOGO"]){?>
						<img class="logo_distr" src="<?=$company["LOGO"]?>" alt="<?=$company["NAME"]?>">
						<?} else {?>
							<div></div>
						<?}?>
						<?if(!$_GET["id"]):?>
						<a href="/test/?id=<?=$company["ID"]?>">
							<svg xmlns="http://www.w3.org/2000/svg" class="bi bi-geo-alt-fill" viewBox="0 0 16 16">
							<path d="M8 16s6-5.686 6-10A6 6 0 0 0 2 6c0 4.314 6 10 6 10m0-7a3 3 0 1 1 0-6 3 3 0 0 1 0 6"/>
							</svg>
						</a>
						<?endif;?>
					</div>
					<div class="textUp color_orange">
						<b><?=$company["NAME"]?></b>
					</div>

					<?if($company["ATT_WEB"] && $company["ATT_WEB"] != '-'){?>
					<div>
						<svg xmlns="http://www.w3.org/2000/svg" class="bi bi-pc-display-horizontal svg" viewBox="0 0 16 16">
                        <path d="M1.5 0A1.5 1.5 0 0 0 0 1.5v7A1.5 1.5 0 0 0 1.5 10H6v1H1a1 1 0 0 0-1 1v3a1 1 0 0 0 1 1h14a1 1 0 0 0 1-1v-3a1 1 0 0 0-1-1h-5v-1h4.5A1.5 1.5 0 0 0 16 8.5v-7A1.5 1.5 0 0 0 14.5 0zm0 1h13a.5.5 0 0 1 .5.5v7a.5.5 0 0 1-.5.5h-13a.5.5 0 0 1-.5-.5v-7a.5.5 0 0 1 .5-.5M12 12.5a.5.5 0 1 1 1 0 .5.5 0 0 1-1 0m2 0a.5.5 0 1 1 1 0 .5.5 0 0 1-1 0M1.5 12h5a.5.5 0 0 1 0 1h-5a.5.5 0 0 1 0-1M1 14.25a.25.25 0 0 1 .25-.25h5.5a.25.25 0 1 1 0 .5h-5.5a.25.25 0 0 1-.25-.25"></path>
                        </svg>
						<a href="<?=$company["ATT_WEB"]?>" target="_blank">сайт: <?=$company["ATT_WEB"]?></a>
					</div>
					<?}?>

					<?if($company["ATT_PHONE"] && !empty($company["ATT_PHONE"]) && $company["ATT_PHONE"][0] != '-'){?>
					<div class="disp_flex">
						<svg xmlns="http://www.w3.org/2000/svg" class="bi bi-telephone-forward-fill svg" viewBox="0 0 16 16">
                            <path fill-rule="evenodd" d="M1.885.511a1.745 1.745 0 0 1 2.61.163L6.29 2.98c.329.423.445.974.315 1.494l-.547 2.19a.68.68 0 0 0 .178.643l2.457 2.457a.68.68 0 0 0 .644.178l2.189-.547a1.75 1.75 0 0 1 1.494.315l2.306 1.794c.829.645.905 1.87.163 2.611l-1.034 1.034c-.74.74-1.846 1.065-2.877.702a18.6 18.6 0 0 1-7.01-4.42 18.6 18.6 0 0 1-4.42-7.009c-.362-1.03-.037-2.137.703-2.877zm10.761.135a.5.5 0 0 1 .708 0l2.5 2.5a.5.5 0 0 1 0 .708l-2.5 2.5a.5.5 0 0 1-.708-.708L14.293 4H9.5a.5.5 0 0 1 0-1h4.793l-1.647-1.646a.5.5 0 0 1 0-.708"></path>
                        </svg>
						<div>
						<?foreach($company["ATT_PHONE"] as $phone):?>
							<a href="tel:+<?=$phone?>"><?=$phone?></a><br>
						<?endforeach;?>
						</div>
					</div>
					<?}?>

					<?if($company["ATT_EMAIL"] && $company["ATT_EMAIL"] != '-'){?>
					<div>
						<svg xmlns="http://www.w3.org/2000/svg" class="bi bi-envelope-fill svg" viewBox="0 0 16 16">
						<path d="M.05 3.555A2 2 0 0 1 2 2h12a2 2 0 0 1 1.95 1.555L8 8.414zM0 4.697v7.104l5.803-3.558zM6.761 8.83l-6.57 4.027A2 2 0 0 0 2 14h12a2 2 0 0 0 1.808-1.144l-6.57-4.027L8 9.586zm3.436-.586L16 11.801V4.697z"></path>
						</svg>
						<a href="mailto:<?=$company["ATT_EMAIL"]?>"><?=$company["ATT_EMAIL"]?></a>
					</div>
					<?}?>

					<?if($company["ATT_ADDRESS"] && $company["ATT_ADDRESS"] != '-'){?>
					<div class="color_orange">
						Адрес: <?=$company["ATT_ADDRESS"]?> <br>
					</div>
					<?}?>

				</div>
			<?endforeach;?>
		</div>
		<?}?>
	</div>
	<?$APPLICATION->IncludeComponent( 
                                        "bitrix:map.yandex.view", 
                                        "", 
                                        Array( 
                                            "INIT_MAP_TYPE" => "MAP", 
                                            "MAP_DATA" => serialize(array('yandex_scale' => 3, 'PLACEMARKS' => $arPlacemarks)),
                                            "MAP_WIDTH" => "100%", 
                                            "MAP_HEIGHT" => "500", 
                                            "CONTROLS" => array("ZOOM","SMALLZOOM","MINIMAP","TYPECONTROL","SCALELINE"),
                                            "OPTIONS" => array("ENABLE_SCROLL_ZOOM","ENABLE_DBLCLICK_ZOOM","ENABLE_RIGHT_MAGNIFIER","ENABLE_DRAGGING"), 
                                            "MAP_ID" => "" 
                                    )
                                    );?>
</div>

<?endif;?>
<br><br>


<?require($_SERVER["DOCUMENT_ROOT"]."/bitrix/footer.php");?># bitrix
