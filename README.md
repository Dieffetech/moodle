**COME AGGIORNARE MOODLE**

Andare al link <https://download.moodle.org/releases/latest/> e scaricare l’ultima versione stabile

![](docs/image.001.png)

Scompattare il file in una cartella. 



Aprire il file login/index.php e alla riga 222 circa ci dovrebbero essere queste righe

![](docs/image.002.png)

Che vanno sostituite con queste:
```
// sets the username cookie

if (!empty($CFG->nolastloggedin)) {

    // do not store last logged in user in cookie

    // auth plugins can temporarily override this from loginpage\_hook()

    // do not save $CFG->nolastloggedin in database!

} else if (empty($CFG->rememberusername) or ($CFG->rememberusername == 2 and empty($frm->rememberusername))) {

    // no permanent cookies, delete old one if exists

    set\_moodle\_cookie('');
} else {

    set\_moodle\_cookie($USER->username);
}

$urltogo = core\_login\_get\_return\_url();
```
![](docs/image.003.png)

queste righe di codice le trovi anche nel file login/index.php in allegato

poi vai alla riga 275 circa e trovi delle righe tipo questa:

![](docs/image.004.png)

E dopo la variabile $SESSION incolli le righe sotto sostituendo la funzione redirect
```
if (!empty($\_REQUEST['scoid'])) {
    redirect(new moodle\_url('/mod/scorm/player.php?display=popup&scoid='.$\_REQUEST['scoid'].'&a='.$\_REQUEST['cm']));
} else {
    redirect(new moodle\_url(get\_login\_url(), array('testsession'=>$USER->id)));
}
```

Verrebbe cosi:

![](docs/image.005.png)

queste righe di codice le trovi anche nel file login/index.php in allegato



poi vai alla riga 376 circa dove c’è echo $OUTPUT->header();

![](docs/image.006.png)

E sostituisci tutto il contenuto sotto $OUTPUT con il codice sotto:

```
if(!empty($\_REQUEST['hash']))

{

if (isloggedin() and !isguestuser()) {

    // prevent logging when already logged in, we do not want them to relogin by accident because sesskey would be changed



    $logout = new moodle\_url('/login/logout.php', array('sesskey'=>sesskey(),'loginpage'=>1,'hash'=>$\_REQUEST['hash']));

    redirect($logout);



} else {

    $loginform = new \core\_auth\output\login($authsequence, $frm->username);

    $loginform->set\_error($errormsg);

    echo $OUTPUT->render($loginform);

}

}

else

{

if (isloggedin() and !isguestuser()) {

    // prevent logging when already logged in, we do not want them to relogin by accident because sesskey would be changed

    echo $OUTPUT->box\_start();

    $logout = new single\_button(new moodle\_url('/login/logout.php', array('sesskey'=>sesskey(),'loginpage'=>1)), get\_string('logout'), 'post');

    $continue = new single\_button(new moodle\_url('/'), get\_string('cancel'), 'get');

    echo $OUTPUT->confirm(get\_string('alreadyloggedin', 'error', fullname($USER)), $logout, $continue);

    echo $OUTPUT->box\_end();

} else {

    $loginform = new \core\_auth\output\login($authsequence, $frm->username);

    $loginform->set\_error($errormsg);

    echo $OUTPUT->render($loginform);

}

}


if(!empty($\_REQUEST['hash']))

{



parse\_str(base64\_decode(urldecode($\_REQUEST['hash'])), $request\_array);

?>

<style>

    #region-main{display:none;}

</style>

<script>

    document.getElementById('login').autocomplete = "off";

    document.getElementById('username').value = "<?php echo $request\_array['username']?>";

    document.getElementById('password').value = "<?php echo $request\_array['password']?>";



    document.getElementById('username').type="text"; 

    document.getElementById('password').type="text"; 

    document.getElementById('username').autocomplete= "off";

    document.getElementById('password').autocomplete= "off";

    document.getElementById('username').readOnly = true;

    document.getElementById('password').readOnly = true;

    var scoid = document.createElement("input");      

    scoid.setAttribute("type", "hidden");

    scoid.setAttribute("name", "scoid");

    scoid.setAttribute("value", "<?php echo $request\_array['scoid']?>");

    document.getElementById("login").appendChild(scoid);

    var cm = document.createElement("input");      

    cm.setAttribute("type", "hidden");

    cm.setAttribute("name", "cm");

    cm.setAttribute("value", "<?php echo $request\_array['cm']?>");

    document.getElementById("login").appendChild(cm);

    document.getElementById('login').submit();

</script>

<?php

}

?>

<script>

document.getElementById('login').autocomplete = "off";

</script>

<?php

//echo $OUTPUT->footer();
```



Apri il file login/logout.php e vai alla riga 41 circa

![](docs/image.007.png)

Sotto aggiungi queste righe

```
// DIEFFE CUSTOM LOGOUT START

if(!empty($\_REQUEST['hash']))

{



$redirect = new moodle\_url('/login/index.php', array('hash'=>$\_REQUEST['hash']));

require\_logout();



redirect($redirect);

}

// DIEFFE CUSTOM LOGOUT EDN
```
Diventa cosi

![](docs/image.008.png)



Apri il file mod/scorm/player.php e vai alla riga 27 circa 
```
$currentorg = optional\_param('currentorg', '', PARAM\_RAW);
```

![](docs/image.009.png)

Sotto currentorg incolla queste righe

```
$scoes = $DB->get\_records\_select('scorm\_scoes', 'scorm = ? AND '.

    $DB->sql\_isnotempty('scorm\_scoes', 'launch', false, true), array($scorm->id), 'sortorder, id', 'id');

if (($sco->organization == '') && ($sco->launch == '')) {

$currentorg = $sco->identifier;

} else {

$currentorg = $sco->organization;

}
```


Vai alla riga 181 circa dove c’è

![](docs/image.010.png)

E dopo l’else incolla 
```
$exitlink = html\_writer::link($exiturl, $strexit, array('title' => $strexit, 'class' => 'btn btn-secondary'));

$PAGE->set\_button($exitlink);
```

![](docs/image.011.png)



Infine in fondo al file incolla queste righe
```
?>

<style>

#scorm\_toc{

    display: none !important;

}

#scorm\_toc\_toggle{

    display: none !important;

}

#page-content h2{

    display: none !important;

}

#page-mod-scorm-player #scormpage div.yui3-u-3-4 {

    width: 100% !important;

}

#scorm\_nav{

    display: none !important;

}

</style>
```


Quindi i file da modificare sono 3:

```login/index.php```

```login/logout.php```

```mod/scorm/player.php```

dopo averli modificati, mi copio questi tre file in una cartella ricostruendo i path 

![](docs/image.012.png)

Prendo il file ```config.php``` dal server e lo copio qua 

![](docs/image.013.png)

A questo punto sono pronto per l’aggiornamento. 

- Mi collego a moodle esempio <https://moodle2.dieffetech.it/admin> 
- faccio la login
- cerco manutenzione nel cerca a sinistra o vai qui <https://moodle2.dieffetech.it/admin/search.php?query=manutenzione>
- manutenzione metti abilita
- salva modifiche
- mi collego in ssh al server
- vado in ```cd /var/www/```
- faccio un backup della cartella dei dati di moodle ```sudo cp -Rf moodle2data /home/dieffetech/moodle2databk```
- faccio un backup del database di moodle ```mysqldump -uroot -p moodle2 > /home/dieffetech/moodle.sql```
- faccio un backup della cartella di moodle ```sudo cp -Rf moodle2.dieffetech.it /home/dieffetech/```
- entro in moodle2.dieffetech.it ```cd moodle2.dieffetech.it/```
- elimino tutto  ```sudo rm -Rf \* sudo rm -Rf \*.\*```
- scarico l’ultima versione stabile di moodle

![](docs/image.001.png)

- lo copio nella cartella sul server (non so perché ma con wget non mi va)
- scompatto ```sudo tar xvf moodle-latest-400.tgz```
- ```sudo cp -rf moodle/\* .```
- ```sudo rm -Rf moodle moodle-latest-400.tgz```
- copiare la cartella moosh dal backup ```sudo cp -Rf /home/dieffetech/moodle2.dieffetech.it/moosh/ /var/www/moodle2.dieffetech.it/```
- ```sudo chown -R www-data:www-data \*```
- ```sudo chown -R www-data:www-data \*.\*```
- ```sudo chmod -R 755 \*```
- ```sudo chmod -R 755 \*.\*```
- incollo nella ```root``` i file che avevo modificato
- ```sudo chmod 740 admin/cli/cron.php```
- andare nuovamente qui <https://moodle2.dieffetech.it/admin/search.php?query=manutenzione> 
- apparirà una schermata che capisce che c’è un aggiornamento, cliccare su continua
- eliminare la cartella ```sudo rm -Rf /var/www/moodle2.dieffetech.it/install``` e ```sudo rm -Rf /var/www/moodle2.dieffetech.it/install.php```
- teoricamente ci sarebbe da aggiornare anche il pacchetto ```/var/www/moodle2.dieffetech.it/moosh``` che è quello che serve per caricare gli scorm su moodle. Dobbiamo fare una prova
![](docs/image.001.png)
- Cliccare su aggiorna database
![](docs/image.001.png)
- Terminato l’aggiornamento disattivare la manutenzione <https://moodle2.dieffetech.it/admin/search.php?query=manutenzione>
- Se tutto ok rimuovere le cartelle di backup


# Query Moodle new endpoint

```
INSERT INTO moodle2.mdl_external_functions (name,classname,methodname,classpath,component,capabilities,services) VALUES
('mod_scorm_delete_all_scorm_sco_tracks','mod_scorm_external','delete_all_scorm_sco_tracks',NULL,'mod_scorm','','moodle_mobile_app');
```
```
INSERT INTO moodle2.mdl_external_services_functions (externalserviceid,functionname) VALUES
(2,'mod_scorm_delete_all_scorm_sco_tracks');
```

