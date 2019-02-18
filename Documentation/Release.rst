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

Version 19.x.x
--------------

Date: <date>

Notes
^^^^^

Features
^^^^^^^^

Bug Fixes
^^^^^^^^^


Version 19.2.0
--------------

Date: 07.02.2019

Bug Fixes
^^^^^^^^^

* #7714 / autocron fails to open logfiles - adjust CWD based on argv(0).


Version 19.1.3
--------------

Date: 28.1.2019

Notes
^^^^^

* If a variable violates a sanitize class, the substituted result can now be configured: a) !!<class>!!, b) '0', c) '', d) '<custom message>'.
* Alerts (based on _link class), might now show only 'ok' (alone, without 'cancel').
* Excel Import - three new options: importNamedSheetsOnly, importSetReadDataOnly, importListSheetNames

Features
^^^^^^^^

* SQL Error / underlining in exception dialog: Add two SQL errors to be underlined in exceptions. Extend to "... in 'order clause'"
* Extend allowed SQL commands in QFQ vars (have been already subscribed in that way in Manual.rst).
* New escape mode 'C' - escapes ':' by '\'.' - useful for variables in variable definition.
* fillStoreVar: Replace setStore() with appendToStore()
* #6914 / Customized typeMessageViolation. Incl. unit tests.
* #7743 / Move error messages to Constants.php. Unit tests use those constants now. 'data-pattern-error' only delivered
  if a 'pattern' is given. 'required' attribute only delivered is set. Detection of 'pattern error' on per QFQ default,
  custom instance wide, per form or per FormElement - per FormElement overwrites other. Move default pattern to constants.
  Make default error text more specific (only if default error text is not explicit set in config, form or form-element)
* #7747 / New options to import Excel files: importNamedSheetsOnly, importSetReadDataOnly, importListSheetNames
* #7684 / Optional hide second button (cancel) in link/question alerts.

Bug Fixes
^^^^^^^^^

* #7743 / Form-Element: Explicit given '0' for MIN or MAX has been interpreted as 'not set'.
* #7702 / Form,Form-Element: Unnecessary evaluation of column 'noteInternal' / 'adminNote'.
* #7695 / Form/URL Forward: pageAlias not substituted.
* #7686 / FormAction/sendmail: uninitialised sendMailAttachment.
* #7685 / Open FormElement from QFQ error message and save modified record: report error about missing {{formId:F}}.
* FormElement.type=select: fixed problem with dynamic update and mode=readonly - list was still selectable.
* Hint to skip leading zeros in version number.

Version 19.1.2
--------------

Date: 17.01.2019

Notes
^^^^^

* Align FormElement labels left/center/right.
* Cleanup FormEditor.
* All SQL commands now are allowed.

Features
^^^^^^^^

* #7620 / Label align left/center/right. Defined by config/form/formelement.
* #7647 / Preparation for Selenium Tests - Implement new token 'A'  to add any custom attribute to '... AS _link'
* Cleanup FormEditor: FormElement as second pill. Hide MultiForm as long as it not active. Rename Various to Layout.
* Manual.rst: Add some tips. Add note how to extend TinyMCE valid_elements.
* Delete composer in Resources dir, run phpunit tests in gitlab pipeline.
* Database.php: allow all SQL commands.

Bug Fixes
^^^^^^^^^

* #7671 / On FormElements, 'fillStoreVar' is now detected and fired at first.
* Fix for checkbox-inline / radio-inline.
* Check for non unique FormElements: multiple empty FE.name (e.g. for FE.type=subrecord) are allowed now.


Version 19.1.1
--------------

Date: 04.01.2019

Bug Fixes
^^^^^^^^^

* #7600 / Path to sendEmail has changed and is updated now.
* #7603 / Fix problem: formEditor.sql broken - missing semicolon. QFQ updates did not played formEditor.sql.
  Unit test for DatabaseUpdate(). Especially that formEditor.sql is running fine.
* #7594 / FE.type=extra: don't name it 'type' or 'id' or 'L' - more detailed error message and an explanation in Manual.rst.
* Config.php: error message about to high session timeout now reports the PHP settings.

Version 19.1.0
--------------

Date: 03.01.2019

Notes
^^^^^

Features
^^^^^^^^

* Session expired: report details about session timestamps
* File not found in FE.type=file: Show more clearly that the pathfilename is only shown when ShowDebugInfo=on.

Bug Fixes
^^^^^^^^^

* #7553 / If a pill is hidden, FE on that one should not be processed during save/update.
* #7573 / Upload: Do FillStoreVar before slaveId.
* #7551 / General error: Access to undefined index.
* #7544 / General error: Download.php / Line: 175.
* #7543 / General error: QuickFormQuery.php / Line: 1364. Happened during upload.
* Download Excel (and all other download types): Content-Disposition header delivered/suppressed in the opposite
  meaning as it should be. Seems to be fixed now.

Version 18.12.3
---------------

Date: 25.12.2018

Features
^^^^^^^^

* Form: Add text 'Record id/Created/Modified' to tooltip of save button.

Bug Fixes
^^^^^^^^^

* #7540 / Form: Upload broken. HelperFile.php: correctRelativePathFileName() broken after refactor of qfq paths.
* #7514 / Report: Broken defaults in _pdf, _file, _link.
* #7538 / Report: Excel.php - access to undefined index.
* #7289 / Report: {{<level>.line.insertId}} - missing for altsql.
* #7539 / Report: Copy to clipboard not reliable. 'Direct' content now correctly encoded. 'Copy file content' fully implemented
  for text files.


Version 18.12.2
---------------

Date: 24.12.2018

Notes
^^^^^

* Version skipped. Build problems in CI queue.

Version 18.12.1
---------------

Date: 22.12.2018

Notes
^^^^^

* Existing installations: update QFQ extension config form-layout.formBsColumns/formBsLabelColumns/formBsInputColumns,formBsNoteColumns.
  old: '12', new: 'col-md-12 col-lg10' resp. smaller values for individual columns.
* New config values:

  * Config/flagProduction: yes/now - differentiate between development und production system. Will be used for
    'throwExceptionGeneralError' too.
  * Debug/throwExceptionGeneralError - shows/hide exception of general errors.

* Renamed config values:

  * SITE_PATH >> sitePath
  * EXT_PATH >> extPath
  * _dbName >> dbName

* Record locking: revert to old behaviour, that a locked record can't be modified by another form, even if the second
  form has modeRecordLock=NONE.

* Autocron: update the system crontab entry to the new path (old 'qfq', new 'Source'):

   .../typo3conf/ext/qfq/Source/external/autocron.php

Features
^^^^^^^^

* #7228 / Show error if form element with same name and class already exists.
* #7494 / Exception 'General Error': disable/enable per config.
* Adapt class paths in composer.json to the newly refactored folder names.
* Add 'col-lg-10' to notes section.
* Added Alert Manager to have better control over the Alerts.
* Config.php: make 'reading dbname' Typo3 version dependent.
* Delete two old composer.json and call new composer in Makefile.
* Excel.php: change autoload.php path to new composer folder.
* Manual.rst: several places where the bs-column description are updated with the new way 'col-md-...' instead of '12'.
  Replace ''{{pageId' with '{{pageAlias'. Replace '... AS _Page' by '... AS _page'. Add 'tablesorter tablesorter-filter'
  to FormEditor example page.
* Move bootstrap.php and BindParamTest.php due to refactoring.
* phpunit.xml: implement const 'PHPUNIT_QFQ'. Store.php: set self::$phpUnit on const 'PHPUNIT_QFQ'
* Refactor: 'extension/qfq/qfq/...' to 'extension/Source/core/...'
* Refactor: Manual.rst update config variables (reorder), add 'qfqLog'. Support.php: formSubmitLog hardcoded to
  fileadmin/protected/log. DOCUMENTATION_QFQ > SYSTEM_DOCUMENTATION_QFQ. Remove config var 'logDir'.
* Refactor: SITE_PATH >> sitePath, EXT_PATH >> extPath, SYSTEM_PATH_EXT >> SYSTEM_EXT_PATH
* Remove report/Define.php, report/Error.php.

Bug Fixes
^^^^^^^^^

* #3464 / Checkboxes now disabled (readonly), even when rendered as Bootstrap. Fixes missing readonly for Template Groups.
* #6467 / Sanitizing a hidden field makes the form unsubmittable. Updated elementupdate for pattern, added "false" as an
  alternative to null.  'element-update' now get's 'pattern=<pattern>|false' on element-update.
* #7089 / FE.type=extra: value already set in SIP store.
* #7223 / Add "-" as allowed characters in filenames.
* #7455 / phpunit: Remove outdated report syntax. Catch exception on trying to open a non existing logfile.
* #7461 / Bug in Doku.
* #7464 / DragAndDrop - Undefined index: ord. Bug in subrecord fixed.
* AbstractException.php: fixed problem with empty $match in sql syntax highlighting.
* Block screen stuck seems fixed.
* Check not to try to number_format() empty string.
* config.qfq.example.php: add missing '>'.
* Fixed broken init in phpunit run. Fixed access to uninitialized var. Throw exception if dndTable or form is missing.
* Fixed missing $formElement[FE_DECIMAL_FORMAT]. Add 'missing primary record' check. Fixed missing fe['id'].
* Fixed problem with '?:' - implemented in 2016.
* Fixed problem with resultset in 'altsql'.
* formEditor.sql, Manual.rst: renamed '{{_dbName...:Y}}' to '{{dbName...:Y}}'.
* Manual.rst: Unify 'dbName*' documentation..
* phpunit: Update fixtures table 'Form', 'FormElement' & 'Dirty' definition. Update LDAP Test - surrounding spaces seems
  to be escaped now by '\20'. Create tests dir outside of source, create phpunit.xml, move some tests and make them work.
  Rename tests directory to Tests
* Record Lock. Revert change in cb2e2a70cfe5c251cffbfce65bdc0899da75a9c5: if a record lock exist, another form, with
  record lock mode=NONE, can't get write access to it. This is what the manual describes.
* Sanitize.php: fixes "Uninitialized string offset: 0" error.
* Save.php: Fixed exception (file size, mime type) for non-existing uploads.
* Store.php: fixed problem with wrong config varname for DB.
* Timeout check throws error since php.ini cookie lifetime is zero during unit test. Fix filepath relative to Typo3 dir
  when running unit test.


Version 18.12.0
---------------

Date: 10.12.2018

Notes
^^^^^

* Config.qfq.php: the variable T3_DB_NAME is not necessary anymore.
* Following SYSTEM_STORE variables renamed. Old: '_dbNameQfq' ,'_dbNameData'. New: 'dbNameQfq' ,'dbNameData'
* New: Bootstrap 'col-lg-10' is defined on every form. On screens greater than 'md' the forms won't be expanded to 100% anymore.

Features
^^^^^^^^

* #3992 / STORE_SYSTEM: dbNameQfq, dbNameData, dbNameT3
* Config.php: read 'dbNameT3' from TYPO3_CONF_VARS or from T3 config file.
* Download.php: get dbNameT3 now from STORE_SYSTEM
* #4906 / Php Session Timeout: can be specified globally in configuration and per form. Affects only logged in FE User.
* #6866 / On logout destroy STORE_USER. Session.php: check if _COOKIE['fe_typo_user'] has changed - yes: clear STORE_USER. Store.php: Rearrange functions.
* #6999 / Bootstrap/Form: define columns for desktop 'col-lg-10'
* #7138 / PDF / single source: deliver without converting
* #7293 / Implement new logging for file upload.
* #7406 / dbinit might contain multiple sql statements now.
* #7407 / MariaDB / Ubuntu 18 complains about missing values if column of type TEXT isn't explicit specified in INSERT. New default for database.init=SET sql_mode = "NO_ENGINE_SUBSTITUTION"
* #7431 / FE.type=afterSave (FE Action): SQL won't report the causing FE.name/id
* #7434 / FE.type=beforeLoad / sqlValidate: Validation message not shown to user
* FormEditor.sql: Switch off 'MySQL strict setting of default values'
* Logger.php: remove UserAgent - that one is logged in FormSubmitLog Table. Add 'cookie' to filter individual actions.
* New css classes for icons: .icon-flipped (mirrors icon), .icon-spin (icon spins once on hover), .icon-spin-reverse (mirror of icon spin).

Bug Fixes
^^^^^^^^^

* #7001 / Error message: if 'modeSql' fails, error message does not contain a reference to the causing FE.
* ErrorHandler.php: raise an error as an exception has been stopped in mid 2017 - reactivated now.
* Fix problem in upload elements: a) in exception FE_ID was not reported, b) fillStoreVar was not fired.
* FormAction.php: throw exception if 'fillStoreVar' selects more than one row.




Version 18.10.3
---------------

Date: 29.10.2018

Notes
^^^^^

* QFQ now recommends ImageMagick instead of GraphicsMagic, due to much better 'auto orient' (JPEG) capabilities.

Features
^^^^^^^^

* #4901 / Implement 'delete split files' if primary file is deleted (record) or replaced against a new one.
* #4901 / Implement 'fileSplitOptions'.
* #4901 / Implement PDF split based on IM 'convert' (jpeg).
* #7012 / Memory limit for file download - replace file_get_content() by readfile().
* #7112 / fabric: configure default color.
* thumbnail: empty path filename is no ignored, instead of throwing an exception. This is the same behaviour as
  building links - no definition means 'no link', and not 'error'.
* Refactor chmod(), unlink(), rename(), rmdir(), copy() to use by QFQ version which throws an exception.

Bug Fixes
^^^^^^^^^

* #3613 - Revert due to bug in dynamic show/hide (Revision 0bb99fd, Revision 77096ca7) -

Version 18.10.2
---------------

Date: 13.10.2018

Features
^^^^^^^^

* #2509 / Render encrypted mailto as single link
* #3281 / Trim form inputs
* #4649 / Add sqlBefore & sqlAfter to sendmail FE, move validate() before sqlBefore()/sqlAfter().
* #4922 / Some minor improvements to the excel import
* #5112 / Add incompatibility warning for encode specialchar and checkType allbut…
* #5450 / MySQL Exception: underline faulty area
* #6596 / uid ExcelExport / PDF
* #6944 / Add double comma SQL Hint


Bug Fixes
^^^^^^^^^

* #3529 / No double urldecode() of GET parameters
* #3613 / Label input note layout
* #3850 / Change set input maxlength
* #4545 / After delete: reload page with original parameters
* #4751 / Add bulleted und numbered lists back to tinyMCE Editor
* #4765 / Extend tooltip visibility for checkboxes and radio buttons
* #6911 / Only fire afterInsert on new record (also fix afterUpdate, beforeInsert,…
* #6929 / Treat single file argument like several file argument (savePdf: copy,…


Version 18.10.1
---------------

Date: 12.10.2018

Features
^^^^^^^^

* #5578 / Safari only handles one filetype in upload dialog.
* #6991 / Optional process 'readonly' FE during save. New FE parameter 'processReadOnly = 0|1'.

Bug Fixes
^^^^^^^^^

* #6880 / Fixed Exceptions with too many details to end user.
* 'Drag and drop' failed due to fillStoreForm requests {{form:S}} which was not necessary for drag and drop.
* Upload: rename 'chmod' to 'chmodFile'. Implement 'chmodDir'. Permissions applied for all new created directories.
* Upload: replace 'rename' with 'copy/unlink'

Version 18.10.0
---------------

Date: 04.10.2018

Features
^^^^^^^^

* #6894 / Upload: chmod for file creation.
* #6886 / Upload: Auto Orient - implementation.
* #6721 / Log switching {{feUser:U}} to qfq.log. Log {{feUser:U}} to sql.log.
* #5458 / Add '{{feUser:U}}' to be shown on exception.
* Manual.rst: Fix missing single tick in special column name '_=<var>'
* Report / Variables copied to STORE_USER via "... AS '_=<varname>'" are now available in STORE_RECORD by {{<varname>:R}}

Bug Fixes
^^^^^^^^^

* #6902 / Drag and drop in subrecords: expect 1 row got nothing.

Version 18.9.2
--------------

Date: 16.09.2018

Notes
^^^^^

* To use the new 'tablesorter' feature, please add 'tablesorter-bootstrap.css', 'jquery.tablesorter.combined.min.js',
  'jquery.tablesorter.pager.min.js', 'widget-columnSelector.min.js' in your Typo3 template record.
  See https://docs.typo3.org/typo3cms/drafts/github/T3DocumentationStarter/Public-Info-053/Manual.html#setup-css-js

    * *Existing* QFQ installations: update your CSS/JS includes! The new tablesorter jquery plugin might break (JS errors
      seen on the console) your installation, if it isn't included!

    * If you use the extension 'UZH Corporate Design Template':

      * Update to the latest version 18.9.0.
      * Add constants in the Typo3 template record 'ext: main': ::

            cd.extra.css.file5 = typo3conf/ext/qfq/Resources/Public/Css/tablesorter-bootstrap.css

            cd.extra.js.file10 = typo3conf/ext/qfq/Resources/Public/JavaScript/jquery.tablesorter.combined.min.js
            cd.extra.js.file11 = typo3conf/ext/qfq/Resources/Public/JavaScript/jquery.tablesorter.pager.min.js
            cd.extra.js.file12 = typo3conf/ext/qfq/Resources/Public/JavaScript/widget-columnSelector.min.js

* STORE_USER: check the examples - great new feature to temporary save user settings.

Features
^^^^^^^^

* #6721 / STORE_USER: variables per browser session
* #6690 / Tablesorter for subrecords

Version 18.9.1
--------------

Date: 15.09.2018

Notes
^^^^^

* Report

  * Type of nesting delimiter in 'report' now limited to '{', '[', '(', '['
  * Hide/reuse report content later: `10.content=hide` and later `1000.head = {{10.content}}`

* New Excel import - copy Excel files directly into a DB table.
* Forms with named primary key different than 'id' are now supported.

Features
^^^^^^^^

* #1261 / Tablesorter, incl. saved sort, combined column sort, filters, pagination
* #3129 / suggestion for subrecord title design
* #4922 / Excel import
* #6300 / Disable preview button for requiredNew forms
* #6314 / HTML Mode & sendMailSubjectHtmlEntity for Forms.
* #6481 / Add custom primary key option to Forms.
* #6645 / better error message for incomplete download as link
* #6650 / Report hide content (line) - use later via a variable. First version - problem with enclosed ticks.
* #6653 / Add save button class/glyphicon/tooltip for submit button
* Thesis code correction
* Update QFQ download URL

Bug Fixes
^^^^^^^^^

* AbtractBuildForm.php: fix problem with subrecords in MultiDB Environment.
* #2340 / Report: Problematic Bracket - form now on, only ''{[(<'' will change the nesting token.
* #3333 / Fixed subquery recognition in reports '10.sql, 10.1.sql ...'
* #4837 / Don't display hidden pills
* #5467 / Fill Record Store when evaluating min/max parameters
* #6621 / Fix shifted subrecord head
* #6646 / Fix broken extraInfoButton for some FEs


Version 18.9.0
--------------

Date: 07.09.2018

Features
^^^^^^^^

* #6357 / Save pdf on server
* #5381 / Stored procedures can be called from QFQ Reports
* #4996 / Log QFQ Update with timestamp.
* #6255 / Inline Report Editing - now with SIP and save.php api

Bug Fixes
^^^^^^^^^
* #6465 / Allow newlines in form action queries (e.g. sqlInsert)
* #4654 / Better FE color highlighting (UX)
* #5689 / Default BS Columns for FormElement match Form setting
* #6484 / Download Links mit css class
* #6576 / download buttons are now rendered disabled with render mode r:3
* Cookie Sitepath: wrong detected in case of API calls.

Version 18.8.2
--------------

Date: 28.8.18

Features
^^^^^^^^

* #6563 / Accept 0 as required.

Bug Fixes
^^^^^^^^^

* DatabaseUpdateData.php: add missed 'on the fly' update for Form.title, changed in FormEditor.sql in 18.8.1
* 6562 / sendmail: redirect all mail - the sender is replaced too.
* Manual.rst: several typos fixed

Version 18.8.1
---------------

Date: 26.08.2018

Features
^^^^^^^^
* #4432 / Every 'form submit' will be logged with raw data.
* #4763 / Render vertical text more stable: '... AS _vertical'
* #4996 / Log QFQ Version update
* #5403 / Tooltip on pills are now supported
* #5876 / Subrecord title of column 'Edit' & 'Delete' are now customizable.
* #6249 / Subrecords can now be reordered via drag and drop.
* #6333 / Add to qfq.log: IP Address, User Agent, QFQ Cookie, FE User

Bug Fixes
^^^^^^^^^

* #6401 / Handle Backticks in sendmail
* #6452 / Empty form title: no title row will be rendered anymore.

Version 18.8.0
--------------

Date: 25.08.2018

Notes
^^^^^

* Excel export
* Copy to clipboard

Features
^^^^^^^^

* #4922 / Excel Export - create Excel sheets from scratch or based on a template.
* #3294 / Improve Typo3 QFQ backend layout. Add sparql syntax highlighting.
* #5878 / Formelement.type=note with #!report - whitespace is trimmed.
* #6314 / HTML Mails enabled by specifying flag 'mode=html'.
* Import/Merge form: A new form 'copyFormFromExt' (see file `copyFormFromExt.sql`) offers a one click import of external
  QFQ forms (incl. renumbering of id's).
* formEditor.sql: resized Form.title from 255 to 511 (requested by IK Tool)
* Drag and Drop now offers the possibility to show the renumbered values.
* Manual.rst: security hints, T3 Setup best practice, text input retype, charactercountwrap.
* Config.qfq: central defaults for DATA_MATCH, DATA_ERROR
* Bootstrap QFQ development: switched from bower to npm only.

Bug Fixes
^^^^^^^^^

* #5843 / File upload: limitation to file extensions are no case insensitive.
* #6247 / Replace deprecated each function
* #6281 / FormElement / column 'note': token '#!report' - STORE_RECORD does not work.
* #6331 / File Upload: Wrong error message if filesize is much too big.
* #6229 / Add QFQ icon to content element and content element wizard
* AbstractException.php: fixed problem with htmlEntities() on link to 'Edit Form' and 'Edit FormElement'.


Version 18.6.1
--------------

Date: 21.06.2018

Notes
^^^^^

* Configuration QFQ: form-config.formDataPatternError. New behaviour: If this field is empty, a more specific default
  message is shown (instead of one message for all situations). Best is to clear this field.

Features
^^^^^^^^

* sqlHint / Note if a query fails and contains some not replaced variables.
* #4438 / Log attack detected: will be logged now to fileadmin/protected/log/qfq.log.
* #4041 / Subrecord: Spalte 'id' automatisch mit '<span class="text-muted">' wrappen.
* #5885 / show 'sql.log' in FE.
* #6121 / Formular: ID per Default in Titel.

Bug Fixes
^^^^^^^^^

* #6283 / Form: hide title frame if empty.
* #4299 / HiddenSelect' into 'master'.
* #6276 / default data-required-error moved to central Config.php.
* #5884 / sql.log by default public - protect against access.
* #6276 / Default check_type messages not shown.
* #6233 / Alert 'Form incomplete' - stays until click - auto disappear would be better.

Version 18.6.0
--------------

Date: 13.06.2018

Notes
^^^^^

* config.qfq.ini migrated to config.qfq.php - the old config.qfq.ini get's `chmod 000`.
* Most of config.qfq.ini migrated to Typo3 / Extension Manager - all but the DB /LDAP credentials.
* Keep in config.qfq.ini: ::

    # Rename DB credentials from DB_<key> to DB_1_<key>, with key = 'NAME|HOST|USER|PASSWORD'
    DB_1_USER = ...
    DB_1_SERVER = ...
    DB_1_PASSWORD = ...
    DB_1_NAME = ...

* NEW: Drag and drop to sort elements! Check the Manual.
* `URL forwardMode`

  * `client` renamed to `auto`.
  * `close` added.

Features
^^^^^^^^

* #6100 / Url Forward Auto: Update Manual.rst. The F.parameter.saveAndClose has been removed again. Mode 'close' can be assigned statically or dynamic.
* #6178 / Input: Step: New option 'step' for FE.parameter.
* Download.php: references to non existing files now reported as missing file, not 'wrong mimetype' anymore.
* #4918 / Drag'n'Drop reorder elements DRAGANDDROP.md, PROTOCOL.md: Doc for "drag'n' drop" implementation.
* dragAndDrop.php: API endpoint DragAndDrop.php: Class for implementing drag'n' drop functionality.
* Link.php: implement new renderMode=8 - returning only the sip. QuickFormQuery.php: New entry point for processing "drag'n' drop".
* #3971 / Form title: new design from form title.

Bug Fixes
^^^^^^^^^

* #5077 / Dynamic Update & FE.type=required: Server fixed -

a) dynamic calculated modeSql respected,
b) formModeGlobal=requiredOff respected,
c) dynamic FE with mode='hidden' are not saved anymore.

* #6176 / Icon not aligned when error text: Buttons now wrapped in one 'input-group'.
* Manual.rst: reformat autocron QFQ code.
* #5880 / Skip Error Message during dynamicUpdate.
* #5870 / Missing file config.qfq.ini: Clean QFQ message.
* #5924 / config.qfq.ini/LocalConfiguration.php: several places in formEditor.sql still contained the 'dbIndex...'.
* #6168 Configuration language setting ignored: Form and FormElement editor still used uppercase config values for
  language configuration. Updated to the new camel case notation.
* #5890 / config.qfq.ini is public readable. Renamed file to config.qfq.php. Implement a basic migration assistant to
  copy DB credentials to new config.qfq.php. All other values have to be copied to extmanager/qfq-configuration manually.
* #6216 / Oops, an error occurred! Code - unhandled exception will be caught now.


Version 18.4.4
--------------

Date: 28.04.18

Bug Fixes
^^^^^^^^^

* Fix broken ext_emconf.php

Version 18.4.3
--------------

Date: 28.04.18

Bug Fixes
^^^^^^^^^

* Version Number ...04... not supported by TE. Changing naming scheme to omit leading zero.

Version 18.04.1
---------------

Date: 28.04.2018

Bug Fixes
^^^^^^^^^

* config: broken dbIndexQfq, dbIndexData.

Version 18.04.0
---------------

Date: 26.04.2018

Notes
^^^^^

* QFQ marked as 'stable'
* New version numbering: Year.Month.Index
* Manual.rst:

  * AutoCron documentation enhanced.
  * Replace '{{form:S}}' against '{{form:SE}}'.
  * Check list for 'new installations'.
  * Description for config variables enhanced.
  * Details 'how record locking' is done.
  * Details: extraButtonInfo.
  * Replace config.qfq.ini on most places with 'configuration'.

* Path of 'sql.log' / 'mail.log' are now relative to <site path> (not <ext path> as before).

Features
^^^^^^^^

* formEditor.sql: update table cron.
* AutoCron.php: allow https connections with invalid certificate (e.g. 'localhost' is not listed as a valid hostname).
* ext_conf_template.txt: Extension manager configuration setup.

Bug Fixes
^^^^^^^^^

* AutoCron:

  * Update form 'cron' to load/save records in DB_INDEX_QFQ.
  * Fix problem with array in checkForOldJobs().
  * Implement check that re-trigger asynchronous cron jobs are handled correctly.



Version 0.25.15
---------------

Date: 20.03.2018

Features
^^^^^^^^

* Fabric Read Only mockup.

Bug Fixes
^^^^^^^^^

* #5706 / Fixed that problematic characters in 'fileDestination' has not been sanatized.
* Fixed problem with buttons clipping trough alert.
* Client: wrong variable, updated CSS for long errors.

Version 0.25.14a
----------------

Date: 15.03.2018

Features
^^^^^^^^

* Change getMimeType() in Report in case file is missing or `file` beaks: instead to throw an exception, an empty string is returned.
* Updated protocol.md with Alert description.
* Update Status message for save/delete.
* Makefile: 1) remove sonar, add dependency to let update-qfq-doc run. 2) do qfq doc commit inside of the Makefile.
* Client: Changed save timeout from 1500 to 3000.
* Client: removing the blackout screen when modal gets dismissed.
* Client: modal alerts are now blocking everything.
* Manual.rst: fix RST syntax errors.

Bug Fixes
^^^^^^^^^

* #5677-TinyMCE broken - fixed.


Version 0.25.14
---------------

Date: 14.03.2018

Features
^^^^^^^^

* Change notification from 'save: success' to 'Save' and 'delete: success' to 'Delete'.
* DB update: write intermediate QFQ version after every step.

Bug Fixes
^^^^^^^^^

* #5652 / TypeAheadSql: destroyed SQL statement. Fixed broken compare and missing init of $sqlTest.
* #5668 / Fix Broken SIP after login.


Version 0.25.13
---------------

Date: 08.03.18

Features
^^^^^^^^

* AutoCron: Added doc for autocron. Extend AutoCron.php to be MultiDB aware. Update der AutoCron form.
* #4720 / Separate Database for Form & FormElement - Multi DB - fixed problem that 'Quick Edit Form / FormElement' has been broken in MultiDB Setup.
* #5603 / Report: final value of report columns (special column name).
* Fabric / delete now triggers form.changed / emojis work again.
* #5571 / File Upload: save filesize and mimetype automatically in 'upload mode simple',if those columns exist.
* #5423 / two new column names 'filesize', 'mimetype'.
* #5571 / File Upload: save filesize and mimetype.

  * STORE_VARS contains now 'mimeType' and 'fileSize'.
  * sqlBefore and sqlAfter will be fired in Upload Advanced and new in Upload Simple as well.
  * STORE_VARS contains now `filenameOnly`. It can be used in downloadButton=....

Bug Fixes
^^^^^^^^^

* Fabric: Corrected resizing with changed width in editor.
* #5640 / UTF8 encoded strings: MAX LENGTH wrong.

Version 0.25.12
---------------

Date: 18.02.2018

Notes
^^^^^

* New

  * FE.parameter:

    * timeIsOptional
    * enterAsSubmit

  * FE.checkType: Auto
  * Thumbnail rendering. Public or secure.

* Update

  * Multi DB Support: Form & Report

Features
^^^^^^^^

* #5064 / Throw user form exception on invalid date.
* #5308 / TimeIsOptional parameter.
* #5318 / Allow sendmail speaking word token, adjust documentation and fix some typos.
* #5347 / Error Message (Exception): BS colored box for report error messages. Hide technical informations, show it on click.
* #5392 / Violate message with expected date format.
* #5414 / Add checkType Auto, refactor setDefault methods, add smart detection of defaults, extend documentation and rules.
* #3470 / Enter As Submit= on/off - implemented.
* #4437 / violate sanitize message.
* #4542 / input-type-decimal' into 'master'.
* #5298 / Update docs for HTML mails.
* #5333 / Thumbnail: implementation.
* #5425 / Thumbnail: render mode 7 - implemented, rewrite - secure thumbnails are now rendered on first access, not when
  'AS _thumbnail' is called.
* Implemented $dbIndex for Report.
* Implemeted two new STORE_SYSTEM variables: '_dbNameData' and '_dbNameQfq' - those will be automatically filled qfq
  during instantiation QuickFormQuery(). They can be used in Report to easily access the needed DB.
* Increased Formelement.label from 255 to 511.
* Make DB_INIT in config.qfq.ini set by default.
* Notes how to optimize PDF thumbnailing.
* Reformat manual for config.qfq.ini. Copy config.qfq.example.ini to MANUAL.rst. Migrate config defaults from
  setIfNotSet() to array_merge().
* Security: hide $SQL in error messages to regular user.
* New FE.parameter 'inputType'. Can optional be given by webmaster. Additional, the 'type="number"' will be automatically
  set, if the column is of type 'int' or if 'min' and 'max' is numerically.

Bug Fixes
^^^^^^^^^

* #3192 / Fill STORE_RECORD before loading table title.
* #5285 / Make typeAheadPedantic the default.
* #5348 / Exception/Report: level key missing.
* #5367 / Error Report: reworked alerts, updated css for alerts, 'full level' missing, content too much escaped: Fixed
  too much escaping. Form / FormElement Links in error messages now with BS Buttons.
* #5382 / Double quotes in tooltips are now escaped with &quot;.
* #5390 / input validation decimal broken. fixed.
* #5430 / Add unique ID to each radio button for dynamic update.
* Form: 'FormElement' > 'Container' - relied on '{{formId:S}}' even if the FE record already exist - fixed.
* Subrecord Title - now wrapped with <label class='control-label'>.

Version 0.25.11
---------------

Date: 31.01.2018

Notes
^^^^^

* Violating a sanitize class now returns '!!<sanitize class>!!' instead of an empty string.

Features
^^^^^^^^

* #5022 / Variable violates sanatize class: 'msg' instead of empty string - new identifier "!!<sanitize class>!!".
* #4813 / Exception during form load: show 'form edit link' if editor is logged in.
* formEditor.sql: Increas size of Form.title to give more room for SQL statements in.
* Manual.rst: enhance debug tipps.
* #5321 / Plain Link - render mode- only url - implemented.
* Add regex101 link to checkPattern FormEditor.

Bug Fixes
^^^^^^^^^

* Fixed some broken help links in formEditor.sql.
* #5306 / Exception: tt_content_uid wrong - fixed.
* #4303 / Download von doc/docx-Dateien / Download.php - Mime type wird nicht mehr an Dateiname angehängt.
* #5316 / Help on how to send an E-Mail is wrong - several places fixed.
* #5311 / Error Msg SLQ_RAW != SQL_FINAL: Debug message shows outdated SQL_RAW.
* #5309 / min/max broken for date fields. Add min/max attributes to input and date input tag.
* Fabric now detects 'dirty'.
* Manual.rst: Remove broken link to W3C file upload.


Version 0.25.10
---------------

Date: 26.01.2018

Notes
^^^^^

* PROTOCOL.md: update notes.
* Form / Upload: new option 'downloadButton' - if given renders a download button instead of showing the pathFileName.

Features
^^^^^^^^

* #5023 / Fabric: Cut, rotate and enhance uploaded images. Update Manual.
* All FE 'typeahead' fields are set to 'autocomplete="off"'. Respect user setting for 'autocomplete' - if none given
  (mostly), set it for FE 'typeahead' to 'off'.
* #5295 / Upload: check if given QFQ 'maxFileSize' is higher than php.in post_max_size, upload_max_filesize.
* FE.Subrecord: rearranged column order, start columns with uppercase letter.
* New CSS class 'qfq-full-width-left': especially for buttons to become full width.
* New CSS class 'qfq-table-100' - 100% width, with auto width per column. FE.subrecord changed to  'qfq-table-100'.
* #5302 / remove CSS class 'internal / external'.

Bug Fixes
^^^^^^^^^

* #5189 / BCC SendMail Problem - fixed missing double ticks.
* Manual.rst: Update documentation that the default escape type is 'm'. Remove subrecord/list (have been removed long
  time ago). Fix enumeration problem FE.type=radio `classButton`. Add short note for typeahead.js. Remove never
  implemented 'keySemdId...', 'ANREDE'. Fixed typo - replace '\' by '\\' on most places (not in code sections).
  More generic SQL to extract filename from pathFileName. Fixed several phinx syntax errors. Add example for
  'recent list'-records.
* Fixed problem with missing 'if note exits' in CREATE TABLE `Split`.
* #5030 / Manual.rst: Fixed example with XSS vulnerability.
* #5275 / typeahead.bundle.min.js missing in Manual.rst: fixed.
* FormEditor: 'typeahead' for column 'name' fixed. Attention: only succeed if DB_1_NAME is the final DB (mostly given).
* #5048 / Default value NULL in pathFileName breaks uploads.
* #5028 / Links im FormularEditor zeigen ins Leere (Fehlende Ziel-Anker) - fixed.
* Make readonly BS radio buttons non-selectable.

Version 0.25.9
--------------

Date: 17.12.2017

Features
^^^^^^^^

* #5133 / sendmail: subject and body html entity decode: Introduce options for 'subject' and 'body' to switch on/off HTML encoding / decoding
* Manual.rst: Add notes to QFQ installation, wkhtml problems, paragraph on 'sendEmail' Html2Pdf.php: Add error codes and a hint on wkhtml fails.
* Reformat table qfq-letter.css.less: redefined h1, letter-receiver.

Bug Fixes
^^^^^^^^^

* Bug in sendEmail: invalid SSL_version specified at /usr/share/perl5/IO/Socket/SSL.pm line 575. Patch for sendEmail
  (see https://unix.stackexchange.com/a/68952).


Version 0.25.8
--------------

Date: 11.12.2017

Features
^^^^^^^^

* #5080 / Dynamic PDF Letter.
* #5083 / Bodytext / Report: join lines without spaces.

Bug Fixes
^^^^^^^^^

* Fix problem with commit from 8.12.17 / Store.php: appendToStore.php stopped working - 'report' failed to replace
  '{{<column>:R}}'.
* Store.php: fix problem with empty 'appendToStore()' call.

Version 0.25.7
--------------

Date: 07.12.2017

Notes
^^^^^

* Report: parameter in '... AS _sendmail' needs token now - position dependent is removed now.
* Report: parameter 'a:' in '... AS _sendmail' replaced by 'F:' to be compatible with downloads. Do not separate files by comma.
* Manual: most occurences of 'U:' replaced by 'p:' - same meaning.

Features
^^^^^^^^

* #4255 / Attachments for emails implemented.

Bug Fixes
^^^^^^^^^

* Bug - PHP Warning: Declaration of qfq\BuildFormTable::head() should be compatible with
  qfq\AbstractBuildForm::head($mode = qfq\FORM_LOAD) - fixed.


Version 0.25.6
--------------

Date: 03.12.2017

Notes
^^^^^

Bigger changes in update form after save/dynamic update.

Bug Fixes
^^^^^^^^^

* #4865 / Pill Dynamic Updates Show / Hide.
* #5031 / Missing details in DbException: New definition of SYSTEM_SHOW_DEBUG_INFO: even after config.qfq.ini is parsed
  and SIP Infos has been read - if there is no BE User logged in, the value stays on 'auto' (earlier it has been replaced
  to 'no'). Staying on 'auto' keeps the information that replacing is still open and not replaced means 'no'-BE User logged in.
* #5016 / Loose checkbox value on save - Dirty workaround - better solution necessary.
* #5017 / STORE_RECORD used in FormElement and via '#!report' - save & restore STORE_RECORD.
* #5004 / FormElement with state 'ReadOnly' will be saved with empty value - existing values will be overwritten - fixed.
* 'element-update' for type 'UPLOAD seems to make trouble. Exclude it like 'SELECT'.


Version 0.25.5
--------------

Date: 23.11.17

Bug Fixes
^^^^^^^^^

* #4771: Workaround which switches off updates to SELECT lists, if they are part of a Multi-FE-Row.


Version 0.25.4
--------------

Date: 22.11.2017

Notes
^^^^^

* New keywords / features in report:

  * `altsql`: Fire the query if there is no record selected in `sql`. Shown after `althead`.
  * `shead`: Static head - will always be shown (before `head`), independent of sql selects records or not.
  * `stail`: Static tail - will always be shown (after `tail`), independent of sql selects records or not.

Features
^^^^^^^^

* #2948 / altsql, shead, stail - new directives in Report.
* #4255 / Attachments fuer 'Email'. Static files can be attached to mails.

Bug Fixes
^^^^^^^^^

* #4980 / Variables in Report: a) nested not replaced, b) 'rbeg' not replaced, c) missing unit tests.


Version 0.25.3
--------------

Date: 19.11.2017

Notes
^^^^^

* Report:

  * Special column name 'sendmail': the old way of position dependent parameter are deprecated. Instead use the new
    defined token. See https://docs.typo3.org/typo3cms/drafts/github/T3DocumentationStarter/Public-Info-053/Manual.html#column_sendmail

  * Every row is now merged in STORE_RECORD. Inner SQL statement can now retrieve outer values via STORE_RECORD.
    E.g. `{{column:R}}`. No more level keys!

* The config.qfq.ini directive `VAR_ADD_BY_SQL` is replaced by `FILL_STORE_SYSTEM_BY_SQL_?`. Up to 3 statements are possible.

Features
^^^^^^^^

* Report / sendmail: control via token.
* #4967 / config.qfq.ini: Rename 'VAR_ADD_BY_SQL' to 'FILL_STORE_SYSTEM_BY_SQL_1'. Handle up to 3 FILL_STORE_SYSTEM_SQL_x.
  Implement an optional error message together with a full stop.
* #4766 / Set STORE_RECORD in Report per row.

Bug Fixes
^^^^^^^^^

* #4966 / Variable {{feUser:T}} is not available in config.qfq.ini `FILL_STORE_SYSTEM_?` - changed ordering of store
  initialization. Now: TCY...
* #4944 / Delete: broken when using 'tableName' (instead of form).
* #4904 / Undefined Index: DIRTY_FE_USER - PHP problem that constants cant be replaced inside of single ticks. Fixed.
* #4965: insert path to QFQ cookie/session, to make usage of multiple QFQ installation on one host possible.


Version 0.25.2
--------------

Date: 8.11.2017

Notes
^^^^^

* Starting with this release, the default escape mode is 'm' (mysql_real_escape).

Features
^^^^^^^^

* Default Escape Type changed from 's' to 'm'. DatabaseUpdateData.php: removed the DB update from last commit - not necessary.
  Config.php: New default 'm' Evaluate.php: Respect EscapeTypeDefault in form definition.
  QuickFormQuery.php: Replace 'EscapeTypeDefault' in form defintion very early.
* #4049 / QFQ Variables '{{...}}' might now contain a default value.
* If 'pageAlias:T' is empty, take 'pageId:T'.

Bug Fixes
^^^^^^^^^

* #4836 / Multiple entries in table after several clicks on save. Created a saveInProgress Variable.
* Replaced latest project homepage URL in Manual.rst.
* Fix example SQL for periodId in config.qfq.ini in Manual.rst.
* Remove multiple header 'RELEASE' - there has to be only one.

Version 0.25.1
--------------

Date: 3.11.2017

Bug Fixes
^^^^^^^^^

* #4857 / broken (stale) download: multiple 'u:..' or 'u:...'.
* #4212 / Broken JSON on response to save new record 'Unknown index' fixed by isset().

Version 0.25.0
--------------

Date: 10.10.2017

Notes
^^^^^

* The config.qfq.ini directives DB_USER, DB_NAME, DB_HOST, DB_PASSWORD are replaced by DB_1_USER, DB_1_NAME, DB_1_HOST,
  DB_1_PASSWORD. The old directives are still used, as long as the new directives does not exist.

* New config.qfq.ini directives: DB_INDEX_DATA, DB_INDEX_QFQ.

Features
^^^^^^^^
* #4720 / Separate database handles for QFQ 'form' and QFQ 'data' - 'Form' might  now load/save from forign database/host/user.


Version 0.24.0
--------------

Date: 09.10.2017

Notes
^^^^^

* Change Remove SYSTEM_SECURITY_ABSOLUTE_GET_MAX_LENGTH - makes no sense to hardcode an upper limit.

Features
^^^^^^^^

* Feature Manual.rst: Doc updated for latest subrecord column special names.
* Feature AbstractBuildForm.php: new function subrecordHead(). Replaced several hard coded subrecord column names against constants.
* Feature #4456 / formModeGlobal=requiredOff - update Manual.rst.
* Feature #4606 / _link: qualifier to render bootstrap button - fix unit tests for tooltip. Add tooltip to button/text,
  even if there is no link. Implement token  'b:...' for link class. Manual is updated. Open: `pageX` should be recoded
  to use the new 'b:' instead of hardcoed behaviour to render a button.
* Feature: Upload Button - wrapped with Bootstrap Button. New option 'fileButtonText' to specify a button text.
* Feature #3752 / Pills auf mode|modeSql=hidden|readonly setzen - implemented during 'form load' (not dynamic update).
* Feature: Neu wird nach dem Speichern das Formular nochmal komplett geladen. Das ist wichtig um die durch aftersave
  geaenderten Records in die Formularelemente zu bekommen.
* Feature #4511 / Form: URL Forward - mode dynamic computed - more generic implementation.

Bug Fixes
^^^^^^^^^

* Bug #4731 / Dynamic Update: load(post) triggers 'check required' - makes no sense during filling a form - fixed.
* Bug #4730 / InvalidDate-00-00-2000 FE.type=date - detection of empty date was broken for '00.00.0000'.
* Bug Fixed problem in subrecord when no record is selected.
* Bug #4620 / Easy Fix: saveButtonText / closeButtonText Formatierung.

Version 0.23.1
--------------

Date: 23.9.2017

Bug Fixes
^^^^^^^^^

* #4620 / Easy Fix: saveButtonText / closeButtonText Formatierung.


Version 0.23.0
--------------

Date: 17.09.2017

Features
^^^^^^^^

* #3752 / Pills auf mode|modeSql=hidden|readonly setzen - implemented during 'form load' (not dynamic update).

Bug Fixes
^^^^^^^^^

* #4548 /Template Group: 'form-update' broken - Broken Redirect after Save - Broken same HTML ID for FE copies in a
  template group.
* #4548 /Template Group: 'form-update' broken - max tg element value/index shown after save instead of last user
  supplied value, but save is ok. Neu wird nach dem Speichern das Formular nochmal komplett geladen. Das ist wichtig um
  die durch aftersave geaenderten Records in die Formularelemente zu bekommen.

Version 0.22
------------

Date: 14.09.2017

Notes
^^^^^

* Form Editor: element 'forwardPage' is static again (no dynamic update) - see features in #4511.

Features
^^^^^^^^

* #4511 / Form: URL Forward - mode dynamic computed.

Bug Fixes
^^^^^^^^^

* #4512 | SIP URL does not respect anker token '#'- fixed PLUS: L and type _GET Params included in links which contain a SIP (regular links still open).
* #4508 / Form: during Save with FE with 'report'-Note/Values an exception is thrown - report does not expect, to be called without typo3 - but this is the case during save and generating the JSON.
* #4021 / "required" asterik does not handle multi column labels correctly.
* #4423 / Date inputs with readonly: label is grey.
* Empty date might create '2001-00-00'.
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

* #4431 / FE.type=note: QFQ Report Syntax in 'FE.value' and 'FE.note'.
* #4456 / formModeGlobal=requiredOff - Switches FormElement.mode=required to 'show' for all FE of the current Form.
* #4356 / Form: required parameter - split between 'New' & 'Edit'.

Version 0.20.0
--------------

Changes
^^^^^^^

* New configuration value EXTRA_BUTTON_INFO_POSITION in config.qfq.ini.

Features
^^^^^^^^

* #4386 Fuer GRC: Optional Info Button bei 'input' wie bei 'textarea' - EXTRA_BUTTON_INFO_POSITION=below.
* #4429 / subrecord: new FE parameter 'subrecordTableCass' - a custom class for the subrecord table might be specified.
* #4428 / subrecord: mode=readonly.
* #4421 / subrecord: column of the sql1 row should go into the edit link - implemented.
* #4399 / Do not render '_pdf' when r:5 or empty string.

Bug Fixes
^^^^^^^^^

* #4396 / FE: Justify DATE and TIME in case it's DATETIME on a non primary table.
* #2414 / Deaktivieren von Option 'new' bei subrecord hat keine Folge.
* #4426 / Subrecord: mode=hidden - still shown.
* #4425 / Subrecords: Table head is not wrapped in <thead>.
* #4331 / SQL Statement 'REPLACE' not fired - Keyword missing in list of SQL Keywords.


Version 0.19.7
--------------

Changes
^^^^^^^

* #4306 / Update Text Subrecord: Please save this record first.

Features
^^^^^^^^

Bug Fixes
^^^^^^^^^

* #4278 / Language: Check that language settings are respectet inside of container / pill / fieldset / templateGroup.
* #4310 / Fixed error where custom values wouldn't be saved, nor not found for non pedantic.
* #4311 / Record Lock: expired lock wird nicht geloescht bei form reload.
* #4309 / typeahead: allow free entry.

Version 0.19.6
--------------

Features
^^^^^^^^

* #4299 / HTML Element 'Select': Placeholder.
* Changes to the alert generation and added btn-group for multiple buttons.
* Should only show reload button and be modal when the conflict is mandatory.
* #4144 / Close/New: bei acquireLock=false anschliessend keine Nachfrage ob gespeichert werden soll.
* #4120 / Removed Timeout from Dirty Alert Message.
* #4283 / FE.parameter=emptyMeansNull.

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

* #4266 / FormElement Type=Editor: value not saved - fixed.
* #4253 / Record Lock not deleted, when window closes without save.

Version 0.19.3
--------------

Changes
^^^^^^^

* Changing buttons for the dirty Events depending on status.

Bug Fixes
^^^^^^^^^

* #4257 / Dynamic update broken - after changing JSON data structure, update load.php has been missed.

Open
^^^^

* #4253 / Record Lock not deleted when window closes without save.

Version 0.19.2
--------------

Features
^^^^^^^^

* #4250 / autocron: sending mails.
* #4248 / FormElement: TypeAhead fuer den Spaltennamen - Implemented.
* #4144 / Close/New: bei acquireLock=false anschliessend keine Nachfrage ob gespeichert werden soll.
* #4120: Removed Timeout from Dirty Alert Message.

Version 0.19.1
--------------

Features
^^^^^^^^

 * #4172 / record locking: Bob tries to delete a record and get 'status=error': Client should disable 'delete' button.
 * #4185 / Detect modified record.
 * #4143 / New alert removes old alert(s).
 * #4173 / Form: User open's a new tab and press close - alert to inform user that he has to close the tab.
 * #1930, #3980 /  Client: Bei Form Submit den Status 'submit_reason=save|save,close' mitsenden.
 * Implemented: New > Close (save) now closes correctly the current page. Addtional, #1930 has been solved implizit.

Bug Fixes
^^^^^^^^^

 * Bug #4174 / record locking: error message if delete fails due to record locking.
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

* #3881 / Variables: Ex 'keySemId', New 'periodId' (System Store).
* AbstractBuildForm.php: if a datetime / timestamp has the string 'CURRENT_TIMESTAMP' it will be replaced by the current date/time.
* Add new STORE_TYPO3 vars: pageAlias, pageTitle.
* Config.php: cleanup of checking GET variables.
* #3981 / Record Locking.
* Manual.rst: add documentation for record locking.
* Manual.rst: more details about QFQ variables.
* plantuml: sequence diagrams for record locking.

Bug Fixes
^^^^^^^^^

* Bug #4158 / Delete Button im Form fehlen die SIP Parameter.
* Bug #4159 / missing htmlspecialchar_decode() for FE.value supplied content.
* Bug Makefile: fixed unwanted removing of whole 'doc' by 'maintainer-clean' - doc nowadays contains QFQ related
  manually created documentation.
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

* #3953 / Radio buttons: Auswahl nicht angezeigt, wenn per itemList definiert.
* #3982 / Filename Sanatize: remove spaces. Filename not properbly enclosed by double ticks.

Version 0.18.6
--------------

Features
^^^^^^^^

* #3460 / Report: new column types '_striptags, '_htmlentities', '_+Tag'.

Version 0.18.5
--------------

Features
^^^^^^^^

* QuickFormQuery.php: added function to check if there are copyForm paste records.
* Manual.rst, formEditor.sql: add several links in Form 'FormEditor' to the online documentation. The FormEditor now
  contains links to the Online Documentation. Add missing explanations: Required Parameter, Forward.

Bug Fixes
^^^^^^^^^

* #3912 / templateGroup: max. 5 instances are saved.
* Manual.rst: Fixed missing '{{' and '%' in examples.
* #3925 / templateGroup / non primary / delete records: only one at a time.
* Makefile: Artifactory builds fails at chromedriver - temporary remove npm install of chromedriver.

Version 0.18.4
--------------

Bug Fixes
^^^^^^^^^

* #3910 / 'submitButtonText' not shown.

Version 0.18.3b
---------------

Features
^^^^^^^^

* #3906 / Mark required inputs with an asterik.

Bug Fixes
^^^^^^^^^

* #3903 / Copy/Paste form: references inside a record are not updated at all.

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
  * formEditor.sql: New form 'copyForm'. New table 'Clipboard'.


Version 0.18.2
--------------

Changes
^^^^^^^

* #3891 / Dynamic Update: Multiple Elements in a row - single show/hide - First implementation: Show / Hide via
  dynamicUpdate for a second element, wrapped in an own 'col-md' div.

Bug Fixes
^^^^^^^^^

* #3875 / FormElement 'extra': Fehler bei neuen Records.
* #3647 / Dynamic Update: Multiple Elements in a row not updated properly.
* #3890 / upload-FormElement: mode 'hide' without effect.
* #3876 / upload-FormElement: verschwindet bei dynamicUpdate.

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
* Config.php: Set defaults for config.qfq.ini SQL_LOG and SQL_LOG_MODE.
* config.qfq.example.ini: Remove DB_NAME_TEST, Add some details about SQL_LOG, add example for TECHNICAL_CONTACT.
* Session.php: Activate cookie_httponly for QFQ cookies.
* Html2Pdf.php:  recode setting of $this->logFile.
* config.qfq.ini: new syntax for SHOW_DEBUG_INFO - [yes,no,auto][,download].
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
* #3863 / New config var 'DB_UPDATE' in config.qfq.ini.

Bug Fixes
^^^^^^^^^

* #3770 / Attack Delay: merge processing to one codeplace. Config.php: new function attackDetectedExitNow(). Sip.php:
  replace local sleep(PENALTY_TIME_BROKEN_SIP) with central function attackDetectedExitNow().
* #3768 / log to sql.log: ip, formname, feuser.
* Store.php: Fix problem that an empty SQL_LOG will be added to SYSTEM_PATH_EXT. Logger.php: do nothing if there is no
  file defined.
* #3751 / download FE_GROUP protected pages failed. Implement additional 'SHOW_DEBUG_INFO = download' to track down
  problems with 'session forwarding'.
* Allow spaces in value when selection <radio> and <select> tags.
* #3812 / extraButtonInfo (extraButtonLock, extraButtonPassword) - Problems in TemplateGroups.
* #3832 / Dynamic Update und Radio buttons - leerer value ( '' ) kann nicht angewählt werden.
* #3612 / Konflikt typeAheadLdap mit dynamic modesql: inputs with typeahead lacks 'dynamicUpdate' via 'modeSql'.
* #3853 / New > Save: Reload des Forms mit neuer SIP und neu erstellter recordId.
* #3854 / Wrong final page: a) New > Save > Close, b) New > Save > Delete, c) New > New
  formEditor.sql: update table 'Form.forwardMode' to ('client', 'no', 'url', 'url-skip-history').
* #2337 / Checkbox: checked/unchecked parameters genügen nicht.
* #2542 / FormElement-Typ 'note' funktioniert nicht mit dynamic update.
* #3863 / DB Update Fails: Expected no record, got 2 rows: SHOW TABLE STATUS WHERE Name='Form'.

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
* LICENSE.txt: Add GPLv3.
* Html2Pdf.php: Add SIP support wkhtmltopdf URLs. Move cookies for wkhtmltopdf from commandline arguments to filebased.
* SessionCookie.php: New class to save current cookies in a file.
* Html2Pdf.php: implemented session forwarding to wkhtmltopdf.
* Session.php: introduced close(). This will unlock the current session. Take care on subsequent calls to reopen primary session again.
* Database.php: Set charset to real_escape_string() functions properly. Proxy for mysqli::real_escape_string().
* Implement honeypot variables to detect bots.
* HTML special char encode all URL GET parameter. This can't be skipped.

Bug Fixes
^^^^^^^^^

* Sip.php: Parameter XDEBUG_SESSUIB_START excluded from GET parameter copied to SIP.
* Manual.rst: add libxrender1 to install by using wkhtmltopdf.
* Download.php: Skip 'pdftk' if there is only one PDF file to concatenate.
* #3615 / download.php: Das Popup schliesst nicht automatisch bei ZIP, im FF, Warnung in der Console, hourglass wobbles.
* Split PHP 'print.php' in a pure API file 'print.php' and a class 'Html2Pdf.php' - the class will be reused by Download.php.
* #3573 / TypeaheadLdap: Prefetch funktioniert nicht.
* #3547 / FE of type 'note' causes writing of empty fields.
* #3546 / Throw of a UserFormException with wrong parameter. Fixed.
* #3545 / Errormessages via API/JSON not displayed.
* #3536 / a) Datum (datetime / timestamp) werden nicht angezeigt, b) Angezeigte Datumsformat String und aktzeptierte Eingabe matchen nicht.
* #3533 / afterSave: sqlUpdate auf child-record ändert xId von Hauptrecord auf 0.

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
  * Change default width on Form for subrecord FormElement.name.
  * Changed input height of 'parameter of FormEditor and FormElementEditor to 8 lines.

* Enlarge placeholder value in FormElement. Old 512, New 2048..
* #3466 / Input Typeahead: optional only allow specified input. Mode: typeAheadPedantic.
* #3465 / Save button: optional 'active after form load': `Form.parameter.saveButtonActive` - if this attribute is set,
  the save button will be enabled directly on form load.
* #3463 / form.mode=readonly. Make a form complete `readonly`. This can be done statically or dynamically via variable (e.g. SIP).
* #3447 / Icons das man im FrontEnd direkt das gewaehlte FormElement im Formulareditor bearbeiten kann. Add checkbox left.
  to the 'EditForm'-Button, to toogle the 'FormElemnt'-Icons. Like the  'Form Edit'-Pencil, the 'FormElement Checkbox'
  is only displayed if the user is logged in BE.
* #3456 / LDAP: with Credentials (e.g. to access 'webpass'). Updated doc for a) config.qfq.ini: LDAP_1_RDN, LDAP_1_PASSWORD,
  b) Form.parameter|FormElement.parameter: ldapUseBindCredentials.

  * ErrorHandler.php: removed details - the end user should not too many details.
  * FormAction.php, Ldap.php, QuickFormQuery.php: implement 'ldapUseBindCredentials'.
  * Ldap.php: set_error_handler() to catch ldap_bind() problems. Always set LDAP_OPT_PROTOCOL_VERSION=3 - this might
    cause problems with som LDAP Servers - we will see.


Bug Fixes
^^^^^^^^^

* 3509 / SELECT Element: value wird nicht selektiert.
* 3502 / TemplateGroups: Checkboxen werden beim ersten Speichern (insert) nicht geschrieben - ein anschliessendes Update
  ist ok.
* 3385 / templateGroup: insert/update/delete non primary records.

  * Non primary record leads to a problem that the default values, given as an array, are not replaced by scalar values. fixed.
  * Update doc how to insert/update/delete non primary templateGroup records.
  * Removed $templateGroupIndex - solved implicit by defining a LIMIT on 'slaveId' . Implemented '%D' (one below %d).
    Implemented FE_SQL_HONOR_FORM_ELEMENTS - reduces unecassary SQL queries.
  * Fill STORE_RECORD during Formload - to read templateGroup records very early. Local copy of `getNativeFormElements()`,
    new `explodeTemplateGroupElements()`

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

 * #3419 / typeAheadSql: Array with only one column or non standard columnnames are not handeld properbly.
   Detection of missing LIMIT implemented.
 * #3425 / Form.parameter, FormElement.parameter: comment handling, trailing & leading spaces
   Manual.rst: commented handling of 'comment character' and 'escaping of leading/trailing spaces'
   Support.php: new funtion handleEscapeSpaceComment().
 * Evaluate.php: parse all F|FE.parameter via handleEscapeSpaceComment(). A leading '#' or ' ' might be escaped by '\'.
 * Saving 'extra' FE in STORE_SIP has been done with inappropiate FE_NAME. Correct is the pure FE_NAME, without any
   extension like recordId. Unessary and broken decoding removed.
 * #3426 / Dynamic Update: Inputs loose the new content and shows the old value.
 * Through fix #2064 the FE.checkType has not been used anymore. This is fixed now.
 * #3433 / templateGroup on primary Record: Values of removed copies are not deleted. The new implementation creates empty
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
     * Display 'name' (internal name) instead of 'label' (shown on the website and might not so usefull as 'name' which
       is nowhere else used than in that drop-down.

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
 * Manual: Added howto prevent <p>-wrap in TinyMCE.
 * TemplateGroup: Add button now disabled if max. number of copies reached.
 * #3414 / QuickFormQuery.php: wrap whole form in 'col-md-XX' - User controls the width of an QFQ form.

Bug Fixes
^^^^^^^^^

 * Dynamic Update has been broken since implementing of 'element-update' (#3180). Now both methods, 'element-update' and 'form-update' should be fine.
 * qfq-bs.css.less: Fixed problem with 'typeahead input elements' not expanded to Bootstrap column width. Changed
   Layout/Design Typeahead drop-down box. Add hoover for the drop-down box with a blue background.
 * AbstractBuildForm.php: #3374 - textarea elements now contains 'maxlength' attribute.
 * BuildFormBootstrap.php: wrapping of optional 'submitButtonText' now done with the 'per form' values.
 * typeahead.php: if there is an exception, the message body is sent as regular 'content' for the drop-down box. At the
   moment this is the only way to transmit any error messages.
 * formEditor.sql: removed all 'maxLength' string values for 'Form' and 'FormElement' forms.
 * Save button becomes active if a templateGroup copy is removed.
 * #3413 / Form ohne Pill hat kein padding am Rand. Fix: if there are no pills, an additinal col-md-12 will be rendered.


Version 0.13
------------

Changes
^^^^^^^

 * Play formEditor.sql.
 * formEditor.sql:

   * Checktype of `Form.name` restricted to `alnumx` (prior `all`).
   * Changed `access` for Form `form` & '`ormElement` from `always` to `sip`.

 * Table `FormElement`:

   * Modified column: `checkType` - new value `numerical`.

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
 * #3063 / Radios and checkboxes optional rendered in Bootstrap layout.
 * Added 'help-box with-errors'-DIV after radios and checkboxes.
 * Respect attribute `data-class-on-change` on save buttons.


Bug Fixes
^^^^^^^^^

 * #2138 / digit sanitize: new class 'numerical' implemented.
 * Fixed recursive thrown exception.
 * #2064 / search of a default value for a non existing tablecolumn returns 'false'.

   * Fixed setting of STORE_SYSTEM / showDebugInfo during API call.

 * #2081, #3180 / Form: Label & note - update via `DynamicUpdate`.
 * #3253 / if there is no STORE_TYPO3 (calls through .../api/ like save, delete, load): use SIP / CLIENT_TYPO3VARS.
 * qfq-bs.css:

   * Alignment of checkboxes and radios optimized.
   * CSS class 'qfq-note' for 'notes' (third column in a form).


Version 0.12
------------

Changes
^^^^^^^

 * Table 'FormElement'
   * New column: rowLabelInputNote:

      ALTER TABLE  `FormElement` ADD `rowLabelInputNote` set('row','label','/label','input','/input','note','/note','/row')
      NOT NULL DEFAULT 'row,label,/label,input,/input,note,/note,/row' AFTER  `bsNoteColumns` ;

   * Modified column: 'type' - new value 'templateGroup':

      ALTER TABLE  `FormElement` CHANGE  `type`  `type` ENUM(  'checkbox',  'date',  'datetime',  'dateJQW',  'datetimeJQW',  'extra',
      'gridJQW',  'text',  'editor',  'time',  'note',  'password',  'radio',  'select',  'subrecord',  'upload',  'fieldset', 'pill',
      'templateGroup',  'beforeLoad',  'beforeSave',  'beforeInsert',  'beforeUpdate',  'beforeDelete',  'afterLoad',  'afterSave',
      'afterInsert',  'afterUpdate',  'afterDelete',  'sendMail' ) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT  'text'

 * formEditor.sql: Added HTML 'placeholder' in FormEditor for bs*Columns.

   * PLAY 'formEditor.sql'.

 * User Input will be UTF8 normalized.

   * INSTALL 'php5-intl' or 'php7.0-intl' on Webserver.

 * Add globalize.js to be included. Needed by jqx-all.js.

   * UPDATE EXISTING TypoScript TEMPLATES of QFQ Installation.

 * Name of variable '_filename' (used in field 'parameter') has changed. Old: '_filename', New: 'filename'.

   * UPDATE `FormElement` SET parameter = REPLACE(parameter, "_filename", "filename")


Features
^^^^^^^^

 * User input will be UTF8 normalized.
 * config.qfq-ini:

   * New configuration values: FORM_BS_LABEL_COLUMNS / FORM_BS_INPUT_COLUMNS / FORM_BS_NOTE_COLUMNS.
   * Comment empty variables - the new default setting is, that empty parameter in config.qfq.ini means EMPTY
     (=parameter is set and will not be overwritten by internal default), not UNDEFINED (overwritten by internal default).

 * FileUpload:

   * Implemented new Formelement.parameter: fileReplace=always - will replace existing files.
   * Multiple / Advanced Upload: new logic implements slaveId, sqlInsert, sqlUpdate, sqlDelete.

 * FormElement.parameter: sqlBefore / sqlAfter fired during 'Form' save for action elements.
 * STORE FORM: variable 'filename' moved to STORE VAR - sanatize class needs no longer specified.
 * STORE VAR: two new variables 'filename' and 'fileDestination' valid during processing of current upload FormElement.
 * Default store priority list changed. Old: 'FSRD', New: 'FSRVD'.
 * CODING.md: update doc for FormElement 'upload' and general 'Form' rendering & save (recursive rendering).
 * User manual:

   * Described form layout options: description for bsLabelColumn, bsInputColumn, bsNoteColumn.
   * Update 'file-upload' doc.
   * Described 3 examples for upload forms.

 * Administrator manual:

   * Add description page.meta...

 * New FormElement (type= 'container') added: 'templateGroup'.

   * FormElement.parameter.tgAddClass | tgAddText | tgRemoveClass | tgRemoveText | tgClass.
   * FormElement.maxSize: max number of duplicates.
   * #3230 / templateGroup: margin between copies. 'tgClass' implemented.

 * Native FormElements:

   * FormElement.parameter.htlmlBefore|htmlAfter - add the specified HTML code before or after the element (outside of any wrapping).
   * #3224, #3231 / Html Tag <hr> als FormElement. >> htmlBefore | htmlAfter.
   * FormElement.parameter.wrapLabel | wrapInput | wrapAfter | wrapRow - if specified, any default wrapping is omitted.
   * FormElement.bsNoteColumns | bsInputColumns | bsNoteColumns - a '0' will suppress the whole rendering of the item.
   * FormElement.rowLabelInputNote - switch on/off rendering of the corresponding system wrapping items.

 * #3232 / Define custom 'on-change' color - used for the save button: Form.parameter.buttonOnChangeClass=...
 * Form.parameter & FormElement.parameter: Lines starting with '#' are treated as comments and will not be parsed.

Bug fixes
^^^^^^^^^

 * User manual:
   * Fixed double include of validator.js in T3 Typoscript template example.
   * Fixed wrong store name SYSTEM: S > Y.
   * Fixed wrong STORE_FORM variable names.
   * Reformat FormElement.parameter description.
   * Styling errors fixed.
 * Use of 'decryptCurlyBraces()' to get better error messages.
 * Skip unwanted parameter expansion during save.
 * Fixed bug with uninitialized FE_SLAVE_ID.
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
 * *FormElement*: Double an input element and validate that the input match: FormElement.parameter.retype=1.
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

 * BuildFormBootstrap.php: added new class name 'qfq-label' to form labels - needed to assign 'qfq-form-right' class.
   Changed wrapping of formelements from 'col-md-8' (wrong) to 'col-md-12'.
 * QuickFormQuery.php: Set default for new F_CLASS_PILL & F_CLASS_BODY.
 * formEditor.sql: New default background color for formElements is blue.
 * qfq-bs.css.less: add classes qfq-form-pill, qfq-form-body, form-group (center), qfq-color-..., qfq-form-right.
 * Index.rst: Add note to hierachy chars. Fixed uncomplete doc to a) bs*Columns, showButton. Add classPill, classBody.
   Rewrote form.paramter.class.
 * QuickFormQuery.php: Button save/ close/ delete/ new - align to right border of form.
 * UsersManual/index.rst: renamed chapter for formelements. Cleanup formelement types. Wrote chapter 'Detailed concept'.
 * QuickFormQuery.php, FormAction.php: '#2931 / afterSave Hauptrecord xId nicht direkt verfügbar' - load master record
   again, after 'action'-elements has been processed.
 * UsersManual/index.rst: Startet FAQ section.
 * config.qfq.example.ini: Added comment where to save config.qfq.ini.
 * UsersManual/index.rst: Rewrite of 'action'-FormElement definition.
 * #2739 / beforeDelete / afterDelete.
 * PROTOCOL.md: update 'delete' description.
 * delete.php: fixed unwanted loose of MSG_CONTENT.
 * Report.php: Fixed double '&&' in building UrlParam.
 * FormAction.php: In case of 'AFTER_DELETE', do not try to load primary record - that one is already deleted.
 * Sip.php: Do not skip SIP_TARGET_URL as parameter for the SIP.
 * #3001 / Report: delete implementieren.
 * Index.rst, Constants.php: reverted parameter '_table' in delete links back to 'table' - Reason: 'form' needs to be
   'form' (instead of '_form') due to many used places already.
 * Sip.php: move SIP_TARGET_URL back to stored inside SIP - it's necessary for 'delete'-links.
 * Report.php, Constants.php: Remove code to handle unecessary 'p:' tag for delete links.
 * Link.php: Check paged / Paged that the parameter r, table and form are given in the right combination.
 * Link.php, Report.php: New '_link' token 'x'. '_paged' and '_Paged' are rendered via Link() class, Link() class now
   supports delete links.
 * QuickFormQuery.php: for modeForm='Form Delete' the 'required param' are not respected - this makes sense, cause these
   parameters typically filled in newly created records.
 * #3076 / Delete Button bei Subrecords erzeugt sporadisch Javascript Exceptions (Webkit: Chrome / Vivaldi) - kein loeschen moeglich.
