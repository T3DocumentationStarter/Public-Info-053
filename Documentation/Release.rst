.. ==================================================
.. Header hierachy
.. ==
..  --
..   ^^
..    ''
..     ;;
..      ,,
..
.. --------------------------------------------------
.. Best Practice T3 reST  https://docs.typo3.org/typo3cms/drafts/github/xperseguers/RstPrimer/
.. External Links: `Bootstrap <http://getbootstrap.com/>`_:
.. Add Images: https://wiki.typo3.org/ReST_Syntax#Images
..
.. -*- coding: utf-8 -*- with BOM.


.. include:: Includes.txt

.. _release:

Release
=======

Version 0.future
----------------

Date: <date>

Notes
^^^^^

Features
^^^^^^^^

Bug Fixes
^^^^^^^^^

Release
=======

Version 0.23.1
--------------

Date: 23.9.2017

Bug Fixes
^^^^^^^^^

* #4620 / Easy Fix: saveButtonText / closeButtonText Formatierung


Version 0.23.0
--------------

Date: 17.09.2017

Features
^^^^^^^^

* #3752 / Pills auf mode|modeSql=hidden|readonly setzen - implemented during 'form load' (not dynamic update)

Bug Fixes
^^^^^^^^^

* #4548 /Template Group: 'form-update' broken - Broken Redirect after Save - Broken same HTML ID for FE copies in a template group.
* #4548 /Template Group: 'form-update' broken - max tg element value/index shown after save instead of last user supplied value, but save is ok. Neu wird nach dem Speichern das Formular nochmal komplett geladen. Das ist wichtig um die durch aftersave geaenderten Records in die Formularelemente zu bekommen.

Release
=======

Version 0.22
------------

Date: 14.09.2017

Notes
^^^^^

* Form Editor: element 'forwardPage' is static again (no dynamic update) - see features in #4511.

Features
^^^^^^^^

* #4511 / Form: URL Forward - mode dynamic computed

Bug Fixes
^^^^^^^^^

* #4512 | SIP URL does not respect anker token '#'- fixed PLUS: L and type _GET Params included in links which contain a SIP (regular links still open).
* #4508 / Form: during Save with FE with 'report'-Note/Values an exception is thrown - report does not expect, to be called without typo3 - but this is the case during save and generating the JSON.
* #4021 / "required" asterik does not handle multi column labels correctly
* #4423 / Date inputs with readonly: label is grey.
* Empty date might create '2001-00-00'
* #4504 / Upload Button: required asterik missing after save - seems to be a problem for every element - should be fixed now.

Version 0.21.0
--------------

Date: 10.09.2017

Notes
^^^^^

* The Form-Editor now has two 'requiredParamter' fields: one for 'New' record and one 'Edit'. Existing settings will be
  automatically copied to both.
* The FormElement-Editor field 'Note' is not anymore a TinyMCE Editor. Instead a regular 'textarea' is used. Main reason
  are incompatibilities between TinyMCE HTML mode and the neede CR/LF linebreaks needed for 'Report' Syntax in the 'note'
  column.

Features
^^^^^^^^

* #4431 / FE.type=note: QFQ Report Syntax in 'FE.value' and 'FE.note'
* #4456 / formModeGlobal=requiredOff - Switches FormElement.mode=required to 'show' for all FE of the current Form.
* Feature #4356 / Form: required parameter - split between 'New' & 'Edit'

Version 0.20.0
--------------

Changes
^^^^^^^

* New configuration value EXTRA_BUTTON_INFO_POSITION in config.qfq.ini

Features
^^^^^^^^

* #4386 Fuer GRC: Optional Info Button bei 'input' wie bei 'textarea' - EXTRA_BUTTON_INFO_POSITION=below
* #4429 / subrecord: new FE parameter 'subrecordTableCass' - a custom class for the subrecord table might be specified.
* #4428 / subrecord: mode=readonly
* #4421 / subrecord: column of the sql1 row should go into the edit link - implemented
* #4399 / Do not render '_pdf' when r:5 or empty string

Bug Fixes
^^^^^^^^^

* #4396 / FE: Justify DATE and TIME in case it's DATETIME on a non primary table.
* #2414 / Deaktivieren von Option 'new' bei subrecord hat keine Folge
* #4426 / Subrecord: mode=hidden - still shown
* #4425/ Subrecords: Table head is not wrapped in <thead>
* #4331 / SQL Statement 'REPLACE' not fired - Keyword missing in list of SQL Keywords


Version 0.19.7
--------------

Changes
^^^^^^^

* #4306 / Update Text Subrecord: Please save this record first

Features
^^^^^^^^

Bug Fixes
^^^^^^^^^

* #4278 / Language: Check that language settings are respectet inside of container / pill / fieldset / templateGroup.
* #4310 / Fixed error where custom values wouldn't be saved, nor not found for non pedantic.
* #4311 / Record Lock: expired lock wird nicht geloescht bei form reload.
* #4309 / typeahead: allow free entry

Version 0.19.6
--------------

Features
^^^^^^^^

* #4299 / HTML Element 'Select': Placeholder
* Changes to the alert generation and added btn-group for multiple buttons.
* Should only show reload button and be modal when the conflict is mandatory.
* #4144 / Close/New: bei acquireLock=false anschliessend keine Nachfrage ob gespeichert werden soll
* #4120 / Removed Timeout from Dirty Alert Message
* #4283 / FE.parameter=emptyMeansNull

Bug Fixes
^^^^^^^^^

* #4281 Prevent save from being clicked multiple times. Save no turns orange when saving.

Version 0.19.5
--------------

Features
^^^^^^^^

* #3790 / Multilanguage: German/ English/ ...

Bug Fixes
^^^^^^^^^
* #4274 / ItemList: escape ',' ':'


Version 0.19.4
--------------

Features
^^^^^^^^

* Feature: Form Paste Records - skip columns during copy if they do not exist on the source side.

Bug Fixes
^^^^^^^^^

* #4266 / FormElement Type=Editor: value not saved - fixed
* #4253 / Record Lock not deleted, when window closes without save.

Version 0.19.3
--------------

Changes
^^^^^^^

* Changing buttons for the dirty Events depending on status

Bug Fixes
^^^^^^^^^

* #4257 / Dynamic update broken - after changing JSON data structure, update load.php has been missed.

Open
^^^^

* #4253 / Record Lock not deleted when window closes without save

Version 0.19.2
--------------

Features
^^^^^^^^

* #4250 / autocron: sending mails
* #4248 / FormElement: TypeAhead fuer den Spaltennamen - Implemented
* #4144 / Close/New: bei acquireLock=false anschliessend keine Nachfrage ob gespeichert werden soll
* #4120: Removed Timeout from Dirty Alert Message

Version 0.19.1
--------------

Features
^^^^^^^^

 * #4172 /record locking: Bob tries to delete a record and get 'status=error': Client should disable 'delete' button
 * #4185 / Detect modified record
 * #4143 / New alert removes old alert(s)
 * #4173 / Form: User open's a new tab and press close - alert to inform user that he has to close the tab
 * #1930, #3980 /  Client: Bei Form Submit den Status 'submit_reason=save|save,close' mitsenden
 * Implemented: New > Close (save) now closes correctly the current page. Addtional, #1930 has been solved implizit.

Bug Fixes
^^^^^^^^^

 * Bug #4174 / record locking: error message if delete fails due to record locking
 * Bug: SQL 'CREATE' implemented as a valid command.

Version 0.19.0
--------------

Changes
^^^^^^^

* bower.json: change bootstrap version number from micro to minor.
* Sip.php: Guarantee that uniqid() is unique at least for the current user.
* Makefile: change installation of phpDocumentor to --alldeps and remove 'phpdoc/'.

Features
^^^^^^^^

* #3881 / Variables: Ex 'keySemId', New 'periodId' (System Store)
* AbstractBuildForm.php: if a datetime / timestamp has the string 'CURRENT_TIMESTAMP' it will be replaced by the current date/time.
* Add new STORE_TYPO3 vars: pageAlias, pageTitle
* Config.php: cleanup of checking GET variables.
* #3981 / Record Locking
* Manual.rst: add documentation for record locking
* Manual.rst: more details about QFQ variables.
* plantuml: sequence diagrams for record locking

Bug Fixes
^^^^^^^^^

* Bug #4158 / Delete Button im Form fehlen die SIP Parameter.
* Bug #4159 / missing htmlspecialchar_decode() for FE.value supplied content.
* Bug Makefile: fixed unwanted removing of whole 'doc' by 'maintainer-clean' - doc nowadays contains QFQ related manually created documentation.
* Bug typeahead.php: An exception catched in typeahead.php has been assigned as array element, instead of a whole array. Fixed.




Version 0.18.7
--------------

Changes
^^^^^^^

* Makefile: 'make bootstrap' udpates all JS Lib packages. Double npm install removed.

Features
^^^^^^^^

* #3947 / Attack detectect: logout current user (only QFQ, FE User still logged in).
* #3959 / Class _link: implement 'any' Bootstrap glyphicons. New token 'G:' for '_link'.

Bug Fixes
^^^^^^^^^

* #3953 / Radio buttons: Auswahl nicht angezeigt, wenn per itemList definiert
* Feature #3982 / Filename Sanatize: remove spaces. Filename not properbly enclosed by double ticks.

Version 0.18.6
--------------

Features
^^^^^^^^

* Feature #3460 / Report: new column types '_striptags, '_htmlentities', '_+Tag'

Version 0.18.5
--------------

Features
^^^^^^^^

* QuickFormQuery.php: added function to check if there are copyForm paste records
* Manual.rst, formEditor.sql: add several links in Form 'FormEditor' to the online documentation. The FormEditor now
  contains links to the Online Documentation. Add missing explanations: Required Parameter, Forward.

Bug Fixes
^^^^^^^^^

* Bug #3912 / templateGroup: max. 5 instances are saved.
* Manual.rst: Fixed missing '{{' and '%' in examples.
* Bug #3925 / templateGroup / non primary / delete records: only one at a time
* Makefile: Artifactory builds fails at chromedriver - temporary remove npm install of chromedriver

Version 0.18.4
--------------

Bug Fixes
^^^^^^^^^
* Bug #3910 / 'submitButtonText' not shown

Version 0.18.3b
---------------

Features
^^^^^^^^

* Feature #3906 / Mark required inputs with an asterik

Bug Fixes
^^^^^^^^^

* Bug #3903 / Copy/Paste form: references inside a record are not updated at all

Version 0.18.3a
---------------

Changes
^^^^^^^

* Copy / Paste form: take care that there is now Form.id=3. This is now the reserved formId for the 'copyForm'.

      UPDATE `FormElement` SET formId=100 WHERE formId=3
      UPDATE `Form` SET `id` = '100' WHERE `Form`.`id` = 3;

Bug Fixes
^^^^^^^^^

Version 0.18.3
--------------

Features
^^^^^^^^

* #3899 / Copy/Paste

  * DatabaseUpdate.php: New table Clipboard, New FE.type='paste', New Form.forwardMode='url-sip' - will be applied for 0.18.3.
  * Store.php: New member in STORE_CLIENT 'CLIENT_COOKIE_QFQ' - might be used to identify current user.
  * formEditor.sql: New form 'copyForm'. New table 'Clipboard'


Version 0.18.2
--------------

Changes
^^^^^^^

* Feature #3891 / Dynamic Update: Multiple Elements in a row - single show/hide - First implementation: Show / Hide via dynamicUpdate for a second element, wrapped in an own 'col-md' div.

Bug Fixes
^^^^^^^^^

* Bug #3875 / FormElement 'extra': Fehler bei neuen Records.
* Bug #3647 / Dynamic Update: Multiple Elements in a row not updated properly.
* Bug #3890 / upload-FormElement: mode 'hide' without effect
* Bug #3876 / upload-FormElement: verschwindet bei dynamicUpdate.

Version 0.18.1
--------------

* Update unit tests

Version 0.18.0
--------------

Changes
^^^^^^^
*  New defaults to FormElement.type=`file` (=Upload).

  * `access` = `application/pdf`
  * `maxFileSize` = `10485760` (10MB)

Features
^^^^^^^^

* Make the image shown for ExtraButtonInfo configureable. Manual.rst: added documentation for new config.qfq.ini
  variables GFX_EXTRA_BUTTON_INFO_INLINE, GFX_EXTRA_BUTTON_INFO_BELOW.
* Manual.rst: description added for `extraButtonLock` ,`extraButtonPassword`, `extraButtonInfo`. Example on how to reuse
  custom vars defined in `config.qfq.ini`. Add three more examples to _pdf. How to access local online documentation.
  Add note about wkhtml only on linux boxes. More wkhtml explanations.
* #3773 / Button: Info / Unlock / ShowPassword: FE_PARAMETER: extraButtonLock / extraButtonPassword / extraButtopnInfo.
* #3769 / Allow specific GET variables longer than SECURITY_GET_MAX_LENGTH. config.php: implemented special handling of
  GET vars, named with '..._<num>'.
* #3766 / SQL_LOG per tt_content record einstellbar machen. Add `sqlLog` and `sqlLogMode` to QFQ tt-content records.
  Add mode 'error' and `none` to sqlLogMode. Manual.rst: Added explanations for SQL_LOG, SQL_LOG_MODE, and tt-content
  pendants sqlLog, sqlLogMode. Update config.qfq.ini to latest attributes.
* Config.php: Set defaults for config.qfq.ini SQL_LOG and SQL_LOG_MODE
* Manual.rst:
* config.qfq.example.ini: Remove DB_NAME_TEST, Add some details about SQL_LOG, add example for TECHNICAL_CONTACT
* Session.php: Activate cookie_httponly for QFQ cookies.
* Html2Pdf.php:  recode setting of $this->logFile.
* config.qfq.ini: new syntax for SHOW_DEBUG_INFO - [yes,no,auto][,download]
* #3711 / Upload Button opens the camera on a smartphone. New attribute 'capture'.
* Database.php: Add SQL keyword 'DROP'.
* #3706 / Check File Upload: a) mime type, b) max file size. Implemented: file upload check for mime type and max file size.
* New 'pageForward'-Mode: 'url-skip-history'.
* FormEditor:

  * Most radio FEs changed from HTML standard to Bootstrap Design.
  * FE 'feIdContainer' will only be shown if there is at least one FE container element.
  * Use dynamicUpdate to hide/show FE 'forwardPage'.
  * FE 'subrecordOption' shown only for FE.type==subrecord.
  * FE 'checkPattern' shown only for FE.checkType=='pattern|min%'.
  * In form 'FormElement' the FE 'formId' removed - not necessary.

* #3568 / Form: fuer alle Buttons (save, close, new, delete) eine optionale class & text konfigurierbar.
* #3569 / Input Optional '0' unterdruecken.
* #3863: New config var 'DB_UPDATE' in config.qfq.ini.

Bug Fixes
^^^^^^^^^

* #3770 / Attack Delay: merge processing to one codeplace. Config.php: new function attackDetectedExitNow(). Sip.php:
  replace local sleep(PENALTY_TIME_BROKEN_SIP) with central function attackDetectedExitNow().
* #3768 / log to sql.log: ip, formname, feuser
* Store.php: Fix problem that an empty SQL_LOG will be added to SYSTEM_PATH_EXT. Logger.php: do nothing if there is no
  file defined.
* #3751 / download FE_GROUP protected pages failed. Implement additional 'SHOW_DEBUG_INFO = download' to track down
  problems with 'session forwarding'.
* Allow spaces in value when selection <radio> and <select> tags.
* #3812 / extraButtonInfo (extraButtonLock, extraButtonPassword) - Problems in TemplateGroups.
* #3832 / Dynamic Update und Radio buttons - leerer value ( '' ) kann nicht angewählt werden
* #3612 / Konflikt typeAheadLdap mit dynamic modesql: inputs with typeahead lacks 'dynamicUpdate' via 'modeSql'.
* #3853 / New > Save: Reload des Forms mit neuer SIP und neu erstellter recordId.
* #3854 / Wrong final page: a) New > Save > Close, b) New > Save > Delete, c) New > New
  formEditor.sql: update table 'Form.forwardMode' to ('client', 'no', 'url', 'url-skip-history').
* #2337 / Checkbox: checked/unchecked parameters genügen nicht.
* #2542 / FormElement-Typ 'note' funktioniert nicht mit dynamic update
* #3863 / DB Update Fails: Expected no record, got 2 rows: SHOW TABLE STATUS WHERE Name='Form'

Version 0.17.0
--------------

Changes
^^^^^^^

* ALTER TABLE  `FormElement` ADD  `encode` ENUM(  'none',  'specialchar' ) NOT NULL DEFAULT  'specialchar' AFTER  `subrecordOption` ;
* UPDATE `FormElement` SET encode='none' WHERE class='native' AND type='editor'

* ALTER TABLE  `Form` ADD  `escapeTypeDefault` ENUM(  '',  's',  'd',  'l',  'L',  'm',  '-' ) NOT NULL DEFAULT  '' AFTER  `permitEdit` ;

* In order to not break functionality of existing forms, it might be necessary (bad for security, good for stability) to
  leave existing forms untouched: `UPDATE Form SET escapeTypeDefault='-'`

* Play formEditor.sql

Features
^^^^^^^^

* New security option `escapeTypeDefault`: will be defined 1) sytem wide in config.qfq.ini, or 2) more specific per
  Form or 3) individually per variable. The later has priority.
* #3544 / Form: view current form - It's now possible to direct view a form, which is currently loaded/edited in the
   FormEditor: Button 'eye' near left of button 'save'.
* #3552 / typeAheadLdapSearchPerToken - webpass kann nicht gleichzeitig nach Vornamen und Nachnamen suchen. Added option
  typeAheadLdapSearchPerToken to split search value in token and OR-combine every search with the individual tokens.
* Download latest QFQ builds and releases: https://w3.math.uzh.ch/qfq/.
* #3218, #3600 / download.php / export: QFQ is now able to create PDFs and ZIPs on the fly. The sources might be
  uploaded PDFs or Websites (local or remote) which will be converted to PDFs.
* Implement 'encode=specialchar' - new option per FormElement which is now the default for every FormElement.
* Sanatize.php: New function urlDecodeArr(). Decode all _GET vars.
* Implemented max GET parameter lenght. Default: 50.
* Implemented new escape class 'mysql' (realEscapeString).
* LICENSE.txt: Add GPLv3
* Html2Pdf.php: Add SIP support wkhtmltopdf URLs. Move cookies for wkhtmltopdf from commandline arguments to filebased.
* SessionCookie.php: New class to save current cookies in a file.
* Html2Pdf.php: implemented session forwarding to wkhtmltopdf.
* Session.php: introduced close(). This will unlock the current session. Take care on subsequent calls to reopen primary session again.
* Database.php: Set charset to real_escape_string() functions properly. Proxy for mysqli::real_escape_string()
* Implement honeypot variables to detect bots.
* HTML special char encode all URL GET parameter. This can't be skipped.

Bug Fixes
^^^^^^^^^

* Sip.php: Parameter XDEBUG_SESSUIB_START excluded from GET parameter copied to SIP.
* Manual.rst: add libxrender1 to install by using wkhtmltopdf.
* Download.php: Skip 'pdftk' if there is only one PDF file to concatenate.
* #3615 / download.php: Das Popup schliesst nicht automatisch bei ZIP, im FF, Warnung in der Console, hourglass wobbles.
* Split PHP 'print.php' in a pure API file 'print.php' and a class 'Html2Pdf.php' - the class will be reused by Download.php
* #3573 / TypeaheadLdap: Prefetch funktioniert nicht
* #3547 / FE of type 'note' causes writing of empty fields.
* #3546 / Throw of a UserFormException with wrong parameter. Fixed.
* #3545 / Errormessages via API/JSON not displayed
* #3536 / a) Datum (datetime / timestamp) werden nicht angezeigt, b) Angezeigte Datumsformat String und aktzeptierte Eingabe matchen nicht.
* #3533 / afterSave: sqlUpdate auf child-record ändert xId von Hauptrecord auf 0

Version 0.16
------------

Changes
^^^^^^^

* Play:

    ALTER TABLE `FormElement` ADD INDEX `feIdContainer` (`feIdContainer`);
    ALTER TABLE `FormElement` ADD INDEX `ord` (`ord`);
    ALTER TABLE `FormElement` ADD INDEX `feGroup` (`feGroup`);

    ALTER TABLE `FormElement` ADD `adminNote` TEXT NOT NULL AFTER `note`;

* Play formEditor.sql

Features
^^^^^^^^

* formEditor.sql:

  * Added 'on update current timestamp'.
  * Add three indexes to formEditor.sql.
  * Column FormElement.adminNote added.
  * Change default width on Form for subrecord FormElement.name
  * Changed input height of 'parameter of FormEditor and FormElementEditor to 8 lines.

* Enlarge placeholder value in FormElement. Old 512, New 2048.
* #3466 / Input Typeahead: optional only allow specified input. Mode: typeAheadPedantic
* #3465 / Save button: optional 'active after form load': `Form.parameter.saveButtonActive` - if this attribute is set,
  the save button will be enabled directly on form load.
* #3463 / form.mode=readonly. Make a form complete `readonly`. This can be done statically or dynamically via variable (e.g. SIP).
* #3447 / Icons das man im FrontEnd direkt das gewaehlte FormElement im Formulareditor bearbeiten kann. Add checkbox left
  to the 'EditForm'-Button, to toogle the 'FormElemnt'-Icons. Like the  'Form Edit'-Pencil, the 'FormElement Checkbox'
  is only displayed if the user is logged in BE.
* #3456 / LDAP: with Credentials (e.g. to access 'webpass'). Updated doc for a) config.qfq.ini: LDAP_1_RDN, LDAP_1_PASSWORD,
  b) Form.parameter|FormElement.parameter: ldapUseBindCredentials

  * ErrorHandler.php: removed details - the end user should not too many details.
  * FormAction.php, Ldap.php, QuickFormQuery.php: implement 'ldapUseBindCredentials'
  * Ldap.php: set_error_handler() to catch ldap_bind() problems. Always set LDAP_OPT_PROTOCOL_VERSION=3 - this might cause problems with som LDAP Servers - we will see.


Bug Fixes
^^^^^^^^^

* 3509 / SELECT Element: value wird nicht selektiert
* 3502 / TemplateGroups: Checkboxen werden beim ersten Speichern (insert) nicht geschrieben - ein anschliessendes Update
  ist ok
* 3385 / templateGroup: insert/update/delete non primary records

  * Non primary record leads to a problem that the default values, given as an array, are not replaced by scalar values. fixed.
  * Update doc how to insert/update/delete non primary templateGroup records.
  * Removed $templateGroupIndex - solved implicit by defining a LIMIT on 'slaveId' . Implemented '%D' (one below %d). Implemented FE_SQL_HONOR_FORM_ELEMENTS - reduces unecassary SQL queries.
  * Fill STORE_RECORD during Formload - to read templateGroup records very early. Local copy of `getNativeFormElements()`, new `explodeTemplateGroupElements()`

* TypeAhead.js: Handle <ENTER> key properly.
* #3462 / FormElement.parameter: requiredList not ok for non numeric content. STORE_FORM had been called without 'sanatize class'.
   Therefore, all non numeric values has been sanatized by default. New: SANATIZE_ALLOW_ALL.
* Corrected error message to use 'itemList' instead of 'itemValues'. Renamed constant too.
* #2542 / FormElement-Typ 'note' funktioniert nicht mit dynamic update. 'Label' and 'note' are fixed - 'value' is still not updated, open.

Version 0.15
------------

Changes
^^^^^^^

 * Play formEditor.sql.

   * Form 'FormElement' failed to display the formtitle of the current form in case of a new FE.
   * Updated subrecord in 'Form' for 'FormElements' - columns 'size' and 'sql1' removed and 'dyn' inserted. Play formEditor.sql.

   *  #3431, Parameter keyword 'typeAheadLdapKeyPrintf' changed to 'typeAheadLdapIdPrintf'.::

        UPDATE FormElement SET parameter = REPLACE(parameter, 'typeAheadLdapKeyPrintf', 'typeAheadLdapIdPrintf')

   * Size 'placeholder' increased::

        ALTER TABLE  `FormElement` CHANGE  `placeholder`  `placeholder` VARCHAR( 2048 ) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT  '';

Features
^^^^^^^^

 * Introduce new config.qfq.ini setting 'EDIT_FORM_PAGE' - by default set to 'form' - the wrench symbol on every Form
   will direct to that page. Fix #3420 / Quicklink 'editform' on form: directs to the current T3 page which might be insufficient.
 * Form 'subrecord' columns: the default width limit of 20 chars are disabled if 'nostrip' is specifed.
 * #3431 / typeAheadSql: columnname 'key' is a reserverd SQL statement - replace by 'id'. Additional the parametername
   'typeAheadLdapKeyPrintf' renamed to 'typeAheadLdapIdPrintf'. By using 'id' instead of 'key' the quoting of the columnname
   is not necessary anoymore.



Bug Fixes
^^^^^^^^^

 * Fix #3419 / typeAheadSql: Array with only one column or non standard columnnames are not handeld properbly.
   Detection of missing LIMIT implemented.
 * #3425 / Form.parameter, FormElement.parameter: comment handling, trailing & leading spaces
    Manual.rst: commented handling of 'comment character' and 'escaping of leading/trailing spaces'
    Support.php: new funtion handleEscapeSpaceComment().
 * Evaluate.php: parse all F|FE.parameter via handleEscapeSpaceComment(). A leading '#' or ' ' might be escaped by '\'.
 * Saving 'extra' FE in STORE_SIP has been done with inappropiate FE_NAME. Correct is the pure FE_NAME, without any
   extension like recordId. Unessary and broken decoding removed.
 * #3426 | Dynamic Update: Inputs loose the new content and shows the old value
 * Through fix #2064 the FE.checkType has not been used anymore. This is fixed now.
 * #3433 | templateGroup on primary Record: Values of removed copies are not deleted. The new implementation creates empty
   fake instances of all copies of templateGroup FormElements. Those are empty. Before save, the submitted form values
   will be expanded with the empty fake templateGroup FormElements and such empty values will be saved.


Version 0.14
------------

Changes
^^^^^^^

 * Play formEditor.sql.

   * All Form & FormEditor input elements now have a maxlength definition of 0, which means take the column definition value.
   * Drop-down list of container assignment:

     * Display 'type' ('pill', 'fieldset', 'templategroup') instead of 'class' (always 'container').
     * Display 'name' (internal name) instead of 'label' (shown on the website and might not so usefull as 'name' which is nowhere else used than in that drop-down.

   * FormElement.placeholder colum width extended to 512:

     ALTER TABLE `FormElement` CHANGE `placeholder` `placeholder` VARCHAR(512) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT ''

 * New class Ldap.php.

Features
^^^^^^^^

 * Typeahead for SQL and LDAP Datasources implemented.
 * formEditor.sql: Changed width of column FormElement.placeholder from 255 to 512. Removed hardcoded 'size' in FormElement 'placeholder'.
 * Character Count: Display a `counter` on input or textarea fields, activated by specifying the formElement.parameter 'characterCountWrap'.
 * Evaluate.php: Two new escape options 'l' and 'L'. Backport of ldap_escape() for PHP <5.6. Multiple escaping for one value now possible.
 * Manual.rst: add some example for TypeAhead and for saving LDAP value.
 * Load foreign values in templatGroups - saving is not implemented yet.
 * Manual: Added howto prevent <p>-wrap in TinyMCE
 * TemplateGroup: Add button now disabled if max. number of copies reached.
 * #3414/QuickFormQuery.php: wrap whole form in 'col-md-XX' - User controls the width of an QFQ form.

Bug Fixes
^^^^^^^^^

 * Dynamic Update has been broken since implementing of 'element-update' (#3180). Now both methods, 'element-update' and 'form-update' should be fine.
 * qfq-bs.css.less: Fixed problem with 'typeahead input elements' not expanded to Bootstrap column width. Changed
   Layout/Design Typeahead drop-down box. Add hoover for the drop-down box with a blue background
 * AbstractBuildForm.php: #3374 - textarea elements now contains 'maxlength' attribute.
 * BuildFormBootstrap.php: wrapping of optional 'submitButtonText' now done with the 'per form' values.
 * typeahead.php: if there is an exception, the message body is sent as regular 'content' for the drop-down box. At the
   moment this is the only way to transmit any error messages.
 * formEditor.sql: removed all 'maxLength' string values for 'Form' and 'FormElement' forms.
 * Save button becomes active if a templateGroup copy is removed.
 * #3413 Form ohne Pill hat kein padding am Rand. Fix: if there are no pills, an additinal col-md-12 will be rendered.


Version 0.13
------------

Changes
^^^^^^^

 * Play formEditor.sql.
 * formEditor.sql:

   * Checktype of `Form.name` restricted to `alnumx` (prior `all`).
   * Changed `access` for Form `form` & '`ormElement` from `always` to `sip`.

 * Table `FormElement`

   * Modified column: `checkType` - new value `numerical`

     ALTER TABLE FormElement MODIFY COLUMN checkType ENUM('alnumx','digit','numerical','email','min|max','min|max date',
       'pattern','allbut','all') NOT NULL DEFAULT 'alnumx'

 * Example Report for `forms` extended by a delete button per row.

Features
^^^^^^^^

 * print.php: offers 'print page' for any local page - create a PDF on the fly (printout is then browser independent).

   * Install `wkhtmltopdf` on the webserver (http://wkhtmltopdf.org/).
   * In config.qfq.ini setup:

        BASE_URL_PRINT=http://www.../
        WKHTMLTOPDF=/opt/wkhtmltox/bin/wkhtmltopdf

 * Check and error report if 'php_intl' is missing.
 * New Checktype 'allow numerical'.
 * Documentation: example for 'radio' with no pre selection.
 * #3063, Radios and checkboxes optional rendered in Bootstrap layout.
 * Added 'help-box with-errors'-DIV after radios and checkboxes.
 * Respect attribute `data-class-on-change` on save buttons.


Bug Fixes
^^^^^^^^^

 * #2138 / digit sanitize: new class 'numerical' implemented.
 * Fixed recursive thrown exception.
 * #2064 / search of a default value for a non existing tablecolumn returns 'false'.

   * Fixed setting of STORE_SYSTEM / showDebugInfo during API call.

 * #2081, #3180 Form: Label & note - update via `DynamicUpdate`
 * #3253, if there is no STORE_TYPO3 (calls through .../api/ like save, delete, load): use SIP / CLIENT_TYPO3VARS.
 * qfq-bs.css:

   * Alignment of checkboxes and radios optimized.
   * CSS class 'qfq-note' for 'notes' (third column in a form).


Version 0.12
------------

Changes
^^^^^^^

 * Table 'FormElement'
   * New column: rowLabelInputNote

      ALTER TABLE  `FormElement` ADD `rowLabelInputNote` set('row','label','/label','input','/input','note','/note','/row')
      NOT NULL DEFAULT 'row,label,/label,input,/input,note,/note,/row' AFTER  `bsNoteColumns` ;

   * Modified column: 'type' - new value 'templateGroup'

      ALTER TABLE  `FormElement` CHANGE  `type`  `type` ENUM(  'checkbox',  'date',  'datetime',  'dateJQW',  'datetimeJQW',  'extra',
      'gridJQW',  'text',  'editor',  'time',  'note',  'password',  'radio',  'select',  'subrecord',  'upload',  'fieldset', 'pill',
      'templateGroup',  'beforeLoad',  'beforeSave',  'beforeInsert',  'beforeUpdate',  'beforeDelete',  'afterLoad',  'afterSave',
      'afterInsert',  'afterUpdate',  'afterDelete',  'sendMail' ) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT  'text'

 * formEditor.sql: Added HTML 'placeholder' in FormEditor for bs*Columns.

   * PLAY 'formEditor.sql'.

 * User Input will be UTF8 normalized.

   * INSTALL 'php5-intl' or 'php7.0-intl' on Webserver.

 * Add globalize.js to be included. Needed by jqx-all.js

   * UPDATE EXISTING TypoScript TEMPLATES of QFQ Installation.

 * Name of variable '_filename' (used in field 'parameter') has changed. Old: '_filename', New: 'filename'

   * UPDATE `FormElement` SET parameter = REPLACE(parameter, "_filename", "filename")


Features
^^^^^^^^

 * User input will be UTF8 normalized
 * config.qfq-ini:
   * New configuration values: FORM_BS_LABEL_COLUMNS / FORM_BS_INPUT_COLUMNS / FORM_BS_NOTE_COLUMNS
   * Comment empty variables - the new default setting is, that empty parameter in config.qfq.ini means EMPTY (=parameter is set and will not be overwritten by internal default), not UNDEFINED (overwritten by internal default).
 * FileUpload:
   * Implemented new Formelement.parameter: fileReplace=always - will replace existing files.
   * Multiple / Advanced Upload: new logic implements slaveId, sqlInsert, sqlUpdate, sqlDelete.
 * FormElement.parameter: sqlBefore / sqlAfter fired during 'Form' save for action elements.
 * STORE FORM: variable 'filename' moved to STORE VAR - sanatize class needs no longer specified.
 * STORE VAR: two new variables 'filename' and 'fileDestination' valid during processing of current upload FormElement.
 * Default store priority list changed. Old: 'FSRD', New: 'FSRVD'.
 * CODING.md: update doc for FormElement 'upload' and general 'Form' rendering & save (recursive rendering).
 * User manual:
   * Described form layout options: description for bsLabelColumn, bsInputColumn, bsNoteColumn
   * Update 'file-upload' doc.
   * Described 3 examples for upload forms.
 * Administrator manual:
   * Add description page.meta...
 * New FormElement (type= 'container') added: 'templateGroup'
   * FormElement.parameter.tgAddClass | tgAddText | tgRemoveClass | tgRemoveText | tgClass
   * FormElement.maxSize: max number of duplicates
   * #3230 templateGroup: margin between copies. 'tgClass' implemented.
 * Native FormElements:
   * FormElement.parameter.htlmlBefore|htmlAfter - add the specified HTML code before or after the element (outside of any wrapping)
   * #3224, #3231 Html Tag <hr> als FormElement. >> htmlBefore | htmlAfter.
   * FormElement.parameter.wrapLabel | wrapInput | wrapAfter | wrapRow - if specified, any default wrapping is omitted.
   * FormElement.bsNoteColumns | bsInputColumns | bsNoteColumns - a '0' will suppress the whole rendering of the item.
   * FormElement.rowLabelInputNote - switch on/off rendering of the corresponding system wrapping items.
 * #3232 Define custom 'on-change' color - used for the save button: Form.parameter.buttonOnChangeClass=...
 * Form.parameter & FormElement.parameter: Lines starting with '#' are treated as comments and will not be parsed.

Bug fixes
^^^^^^^^^

 * User manual:
   * Fixed double include of validator.js in T3 Typoscript template example.
   * Fixed wrong store name SYSTEM: S > Y
   * Fixed wrong STORE_FORM variable names.
   * Reformat FormElement.parameter description.
   * Styling errors fixed.
 * Use of 'decryptCurlyBraces()' to get better error messages.
 * Skip unwanted parameter expansion during save.
 * Fixed bug with uninitialized FE_SLAVE_ID
 * formEditor.sql:
   * The defintion as 'editor' (not text) for FormElement 'note' has been lost - reinserted.
   * Fixed problem while playing SQL query - deleting old FormElements of Formeditor deleted also FormElements of other forms.
 * #3066 / help-text with-error - CSS class 'hidden' will be rendered by default (as long there is no error).
 * Labels are skipped, if FormElement.bsLabelColumns=0.
 * Respect attribute `data-class-on-change` on save buttons.

Version 0.11
------------

Features
^^^^^^^^

 * Added STORE_BEFORE, #3146 - Mainly used to compare old and new values during a form 'save' action.
 * Added 'best practice' for defining and using of 'Central configure values' in UserManual.
 * Added accent characters to sanatize class 'alnumx', #3183.
 * Set default all QFQ send mails to 'auto-submit'.
 * Added possibility to customize error messages ('data-pattern-error', 'data-rquired-error', 'data-match-error',
   'data-error') if validation fails. Customization can be done on global level (config.qfq.ini), per Form or per FormElement.
 * *FormElement*: Double an input element and validate that the input match: FormElement.parameter.retype=1
 * Autofocus in Forms is now supported. By default the first Input Element receives the focus. Can be customized.
 * Added a timestamp in shown exceptions. Usefull for screenshots, send by customer, to find the problem in SQL logfiles.

Bug fixes
^^^^^^^^^

 * Fixed missing docutmentation for FormElement 'note'.
 * Failed SQL queries will now always be logged, even if they do not modify some data.

Version 0.10
------------

Features
^^^^^^^^

 * Implemented Parameter 'extraDeleteForm' for 'forms' and 'subrecords'. Update doc.

Bug fixes
^^^^^^^^^

 * Suppress rendering of form title during a 'delete' call. No one will see it and required parameters are not supplied.
 * In case of broken SQL queries, print them in ajax error message.
 * Remove parameter 'table' from Delete SIP URLs. ToolTip updated.

Version 0.9
-----------

Features
^^^^^^^^

 * FormEditor:
   * design update - new default background color: grey.
   * per form configureable background colors.
   * Optional right align of all form element labels.
   * Added config.qfq.ini values CSS_CLASS_QFQ_FORM_PILL, CSS_CLASS_QFQ_FORM_BODY, CSS_CLASS_QFQ_CONTAINER.

Bug fixes
^^^^^^^^^

 * BuildFormBootstrap.php: added new class name 'qfq-label' to form labels - needed to assign 'qfq-form-right' class. Changed wrapping of formelements from 'col-md-8' (wrong) to 'col-md-12'.
 * QuickFormQuery.php: Set default for new F_CLASS_PILL & F_CLASS_BODY.
 * formEditor.sql: New default background color for formElements is blue.
 * qfq-bs.css.less: add classes qfq-form-pill, qfq-form-body, form-group (center), qfq-color-..., qfq-form-right.
 * Index.rst: Add note to hierachy chars. Fixed uncomplete doc to a) bs*Columns, showButton. Add classPill, classBody. Rewrote form.paramter.class.
 * QuickFormQuery.php: Button save/ close/ delete/ new - align to right border of form.
 * UsersManual/index.rst: renamed chapter for formelements. Cleanup formelement types. Wrote chapter 'Detailed concept'.
 * QuickFormQuery.php, FormAction.php: '#2931 / afterSave Hauptrecord xId nicht direkt verfügbar' - load master record again, after 'action'-elements has been processed.
 * UsersManual/index.rst: Startet FAQ section.
 * config.qfq.example.ini: Added comment where to save config.qfq.ini.
 * UsersManual/index.rst: Rewrite of 'action'-FormElement definition.
 * #2739: beforeDelete / afterDelete.
 * PROTOCOL.md: update 'delete' description.
 * delete.php: fixed unwanted loose of MSG_CONTENT.
 * Report.php: Fixed double '&&' in building UrlParam.
 * FormAction.php: In case of 'AFTER_DELETE', do not try to load primary record - that one is already deleted.
 * Sip.php: Do not skip SIP_TARGET_URL as parameter for the SIP.
 * #3001 Report: delete implementieren.
 * Index.rst, Constants.php: reverted parameter '_table' in delete links back to 'table' - Reason: 'form' needs to be 'form' (instead of '_form') due to many used places already.
 * Sip.php: move SIP_TARGET_URL back to stored inside SIP - it's necessary for 'delete'-links.
 * Report.php, Constants.php: Remove code to handle unecessary 'p:' tag for delete links.
 * Link.php: Check paged / Paged that the parameter r, table and form are given in the right combination.
 * Link.php, Report.php: New '_link' token 'x'. '_paged' and '_Paged' are rendered via Link() class, Link() class now supports delete links.
 * QuickFormQuery.php: for modeForm='Form Delete' the 'required param' are not respected - this makes sense, cause these parameters typically filled in newly created records.
 * Fixed: #3076 Delete Button bei Subrecords erzeugt sporadisch Javascript Exceptions (Webkit: Chrome / Vivaldi) - kein loeschen moeglich.
