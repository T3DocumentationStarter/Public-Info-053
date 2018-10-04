.. ==================================================
.. Header hierarchy
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
.. Add Images: https://wiki.typo3.org/ReST_Syntax#Images ...
..
.. -*- coding: utf-8 -*- with BOM.


.. include:: Includes.txt

.. _general:

General
=======

* Project homepage: https://qfq.io
* Latest releases: https://qfq.io/download
* Development: https://git.math.uzh.ch/typo3/qfq
* Slack: https://qfq-io.slack.com

.. _installation:

Installation
============

The following features are only tested / supported on linux hosts:

* General: QFQ is coded to run on Linux hosts, preferable on Debian derivates like Ubuntu.
* HTML to PDF conversion - command `wkhtmltopdf`.
* Concatenation of PDF files - command `pdftk`.
* Mime type detection for uploads - command `file`.


.. _`preparation`:

Preparation
-----------

Report & Form
^^^^^^^^^^^^^

In PHP 5.x the QFQ extension needs  the PHP MySQL native driver. The following functions are used and are only available with the
native driver (see also: http://dev.mysql.com/downloads/connector/php-mysqlnd/):

* mysqli::get_result (important),
* mysqli::fetch_all (nice to use)

To normalize UTF8 input, the *php5-intl* resp. *php7.0-intl* package is needed by

* normalizer::normalize()

For the `download`_ function, the programs `pdftk` and `file` are necessary to concatenate PDF files.

Preparation for Ubuntu 14.04::

	sudo apt-get install php5-mysqlnd php5-intl
	sudo apt-get install pdftk file                  # for file upload and PDF
	sudo apt-get install inkscape graphicsmagick     # to render thumbnails
	sudo php5enmod mysqlnd
	sudo service apache2 restart

Preparation for Ubuntu 16.04::

	sudo apt install php7.0-intl
	sudo apt install pdftk libxrender1 file pdf2svg  # for file upload, PDF and 'HTML to PDF' (wkhtmltopdf), PDF split
	sudo apt install inkscape graphicsmagick         # to render thumbnails

.. _wkhtml:

wkhtmltopdf
^^^^^^^^^^^

`wkhtmltopdf <http://wkhtmltopdf.org/>`_ will be used by QFQ to offer 'website print' and 'HTML to PDF' conversion.
The program is not included in QFQ and has to be manually installed.

* The Ubuntu package `wkhtmltopdf` needs a running Xserver - this does not work on a headless webserver.

  * Best is to install the QT version from the named website above.
  * In case of trouble with wkhtmltopdf, also install 'libxrender1'.
  * The current version 0.12.4 might have trouble with https connections. Version 0.12.5-dev (github master branch)
    seems more reliable. Please contact the QFQ authors if you need a compiled Ubuntu version of wkhtmltopdf.

In `configuration`_ specify the: ::

    cmdWkhtmltopdf=/opt/wkhtmltox/bin/wkhtmltopdf`.
    baseUrl=http://www.example.com/`.

If wkhtml has been compiled with dedicated libraries (not part of LD_LIBRARY_PATH), specify the LD_LIBRARY_PATH together
with the path-filename: ::

    cmdWkhtmltopdf=LD_LIBRARY_PATH=/opt/wkhtmltox/lib /opt/wkhtmltox/bin/wkhtmltopdf

**Important**: To access FE_GROUP protected pages or content, it's necessary to disable the `[FE][lockIP]` check! `wkhtml`
will access the Typo3 page locally (localhost) and that IP address is different from the client (=user) IP.

Configure via Typo3 Installtool `All configuration > $TYPO3_CONF_VARS['FE']`: ::

   [FE][lockIP] = 0

**Warning**: this disables an important anti-'session hijacking' protection. The security level of the whole installation
will be *lowered*! Again, this is only needed if `wkhtml` needs access to FE_GROUP protected pages & content. As an
alternative to lower the security level, create a separated page subtree which is only accessible (configured via
Typoscript) from specific IPs **or** if a FE-User is logged in.

If there are problems with converting/downloading FE_GROUP protected pages, check `SHOW_DEBUG_INFO = download` to debug.

**Important**: Converting HTML to PDF gives no error message but RC=-1? Check carefully all includes of CSS, JS, images
and so on! Typically some of them fails to load and wkhtml stops running!


HTML to PDF conversion
''''''''''''''''''''''

`wkhtmltopdf` converts a website (local or remote) to a (multi)-page PDF file. It's mainly used in `download`_.

Print
'''''

Different browser prints the same page in different variations. To prevent this, QFQ implements a small PHP wrapper
`print.php` with uses `wkhtmltopdf` to convert HTML to PDF.

Provide a `print this page`-link (replace 'current pageId' )::

	<a href="typo3conf/ext/qfq/qfq/api/print.php?id={current pageId}">Print this page</a>

Any parameter specified after `print.php` will be delivered to `wkhtmltopdf` as part of the URL.

Typoscript code to implement a print link on every page::

	10 = TEXT
	10 {
		wrap = <a href="typo3conf/ext/qfq/qfq/api/print.php?id=|&type=99"><span class="glyphicon glyphicon-print" aria-hidden="true"></span> Printview</a>
		data = page:uid
	}

Send Email
^^^^^^^^^^

QFQ sends mail via `sendEmail` http://caspian.dotconf.net/menu/Software/SendEmail/ - a small perl script without a central
configuration.

By default, `sendEmail` uses the local installed MTA, writes a logfile to `typo3conf/mail.log` and handles attachments
via commandline options. A basic HTML email support is implemented.

The latest version is v1.56, which has at least one bug. That one is patched in the QFQ internal version v1.56p1 (see
QFQ GIT sources in directory 'patches/sendEmail.patch').

Nevertheless, on latest system the TLS support is broken - please check sendEmailProblem_.

The Typo3 sendmail eco-system is not used at all by QFQ.

Thumbnail
^^^^^^^^^

Thumbnails will be rendered via GraphicsMagick (http://www.graphicsmagick.org/) 'convert' and 'inkscape' (https://inkscape.org).
'inkscape' is only used for '.svg' files.

The Typo3 graphic eco-system is not used at all by QFQ.

Usage: `column-thumbnail`_.

Setup
-----

* Install the extension via the Extensionmanager.

  * If you install the extension by manual download/upload and get an error message
    "can't activate extension": rename the downloaded zip file to `qfq.zip` or `qfq_<version>.zip` (e.g. version: 0.9.1).

  * If the Extensionmanager stops after importing: check your memory limit in php.ini.

* Copy/rename the file *<site path>/typo3conf/ext/qfq/config.example.qfq.php* to *config.qfq.php*.
  Configure the necessary settings `configuration`_
  The configuration file is outside the of extension directory, to not loose it during updates.
* When the QFQ Extension is called the first time on the Typo3 Frontend, the file *<ext_dir>/qfq/sql/formEditor.sql* will
  played and fills the database with the *Form editor* records. This also happens automatically after each update of QFQ.
* Configure Typoscript to include Bootstrap, jQuery, QFQ javascript and CSS files.

.. _setup-css-js:

Setup CSS & JS
^^^^^^^^^^^^^^
::

	page.meta {
	  X-UA-Compatible = IE=edge
	  X-UA-Compatible.attribute = http-equiv
	  viewport=width=device-width, initial-scale=1
	}

	page.includeCSS {
		file01 = typo3conf/ext/qfq/Resources/Public/Css/bootstrap.min.css
		file02 = typo3conf/ext/qfq/Resources/Public/Css/bootstrap-theme.min.css
		file03 = typo3conf/ext/qfq/Resources/Public/Css/jqx.base.css
		file04 = typo3conf/ext/qfq/Resources/Public/Css/jqx.bootstrap.css
		file05 = typo3conf/ext/qfq/Resources/Public/Css/qfq-bs.css
		file06 = typo3conf/ext/qfq/Resources/Public/Css/tablesorter-bootstrap.css
	}

	page.includeJS {
        file01 = typo3conf/ext/qfq/Resources/Public/JavaScript/jquery.min.js
        file02 = typo3conf/ext/qfq/Resources/Public/JavaScript/bootstrap.min.js
        file03 = typo3conf/ext/qfq/Resources/Public/JavaScript/validator.min.js
        file04 = typo3conf/ext/qfq/Resources/Public/JavaScript/jqx-all.js
        file05 = typo3conf/ext/qfq/Resources/Public/JavaScript/globalize.js
        file06 = typo3conf/ext/qfq/Resources/Public/JavaScript/tinymce.min.js
        file07 = typo3conf/ext/qfq/Resources/Public/JavaScript/EventEmitter.min.js
        file08 = typo3conf/ext/qfq/Resources/Public/JavaScript/typeahead.bundle.min.js
        file09 = typo3conf/ext/qfq/Resources/Public/JavaScript/qfq.min.js
        file10 = typo3conf/ext/qfq/Resources/Public/JavaScript/jquery.tablesorter.combined.min.js
        file11 = typo3conf/ext/qfq/Resources/Public/JavaScript/jquery.tablesorter.pager.min.js
        file12 = typo3conf/ext/qfq/Resources/Public/JavaScript/widget-columnSelector.min.js

        # Only needed in case FormElement 'annotate' is used.
        file20 = typo3conf/ext/qfq/Resources/Public/JavaScript/fabric.min.js
        file21 = typo3conf/ext/qfq/Resources/Public/JavaScript/qfq.fabric.min.js

	}


.. _form-editor:

FormEditor
----------

Setup a *report* to manage all *forms*:

* Create a Typo3 page.
* Set the 'URL Alias' to `form` (default) or the individual defined value in parameter `editFormPage` (configuration_).
* Insert a content record of type *qfq*.
* In the bodytext insert the following code: ::

    # If there is a form given by SIP: show
    form={{form:SE}}

    # In case indexQfq is different from indexData, set indexQfq.
    dbIndex = {{indexQfq:Y}}

    10 {
        # List of Forms: Do not show this list of forms if there is a form given by SIP.
        # Table header.
        sql = SELECT CONCAT('p:{{pageId:T}}&form=form') as _pagen, '#', 'Name', 'Title', 'Table', '' FROM (SELECT 1) AS fake WHERE '{{form:SE}}'=''
        head = {{'b|p:id={{pageAlias:T}}&form=copyFormFromExt|t:Copy form from ExtForm' AS _link}}
               <table class="table table-hover qfq-table-50">
        tail = </table>
        rbeg = <thead><tr>
        rend = </tr></thead>
        fbeg = <th>
        fend = </th>

        10 {
            # All forms
            sql = SELECT CONCAT('p:{{pageId:T}}&form=form&r=', f.id) as _pagee, f.id, f.name, f.title, f.tableName, CONCAT('form=form&r=', f.id) as _Paged FROM Form AS f ORDER BY f.name
            rbeg = <tr>
            rend = </tr>
            fbeg = <td>
            fend = </td>
        }
    }

.. _install-checklist:

Install Check List
------------------

* Protect the directory `<T3 installation>/fileadmin/protected` in Apache against direct file access.

  * `<T3 installation>/fileadmin/protected/` should be used for confidential (uploaded / generated) data.
  * `<T3 installation>/fileadmin/protected/log/...` is the default place for QFQ logfiles.

* Protect the directory `<T3 installation>/fileadmin` in Apache to not execute PHP Scripts - malicious uploads won't be executed.
* Setup a log rotation rule for `sqlLog`.
* Check that `sqlLogMode` is set to `modify` on productive sites. With `none` you have no chance to find out who changed
  which data and `all` really logs a mass of data.

.. _configuration:

Configuration
-------------

.. _extension-manager-qfq-configuration:

Extension Manager: QFQ Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| Keyword                       | Default / Example                                     | Description                                                                |
+===============================+=======================================================+============================================================================+
| Config                                                                                                                                                             |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| documentation                 | http://docs.typo3.org...                              | Link to the online documentation of QFQ. Every QFQ installation also       |
|                               |                                                       | contains a local copy: typo3conf/ext/qfq/Documentation/html/Manual.html    |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| thumbnailDirSecure            | fileadmin/protected/qfqThumbnail                      | Important: secure directory 'protected' (recursive) against direct access. |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| thumbnailDirPublic            | typo3temp/qfqThumbnail                                | Both thumbnail directories will be created if not existing.                |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| cmdInkscape                   | inkscape                                              | If inkscape is not available, specify an empty string.                     |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| cmdConvert                    | convert                                               | GraphicsMagics 'convert' is recommended.                                   |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| cmdWkhtmltopdf                | /usr/bin/wkhtmltopdf                                  | PathFilename of wkhtmltopdf. Optional variables like LD_LIBRARY_PATH=...   |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| baseUrl                       | http://example.com                                    | URL where wkhtmltopdf will fetch the HTML (no parameter, those comes later)|
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| sendEMailOptions              | -o tls=yes                                            | General options. Check: http://caspian.dotconf.net/menu/Software/SendEmail |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| dateFormat                    | yyyy-mm-dd                                            | Possible options: yyyy-mm-dd, dd.mm.yyyy.                                  |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| Dynamic                                                                                                                                                            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| fillStoreSystemBySql1|2|3     | SELECT s.id AS ...                                    | Specific values read from the database to fill the system store during QFQ |
|                               |                                                       | load. See `fillStoreSystemBySql`_ for a usecase.                           |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| fillStoreSystemBySqlErrorMsg2 | No current period found                               | Only define an error message, if QFQ should stop running                   |
|                               |                                                       | in case of an SQL error or not exact one record.                           |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| Debug                                                                                                                                                              |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| sqlLogMode                    | modify                                                | | *all*: every statement will be logged - this might a lot.                |
|                               |                                                       | | *modify*: log only statements who change data. *error*: log only         |
|                               |                                                       |   DB errors.                                                               |
|                               |                                                       | | *none*: no SQL log at all.                                               |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| sqlLog                        | fileadmin/protected/log/sql.log                       | Filename to log SQL commands: relative to <site path> or absolute. If the  |
|                               |                                                       | directory does not exist, create it.                                       |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formSubmitLogMode             | all                                                   | | *all*: every form submission will be logged.                             |
|                               |                                                       | | *none*: no logging.                                                      |
|                               |                                                       | | See `Form Submit Log page`_ for example qfq code to display the log.     |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| mailLog                       | fileadmin/protected/log/mail.log                      | Filename to log `sendEmail` commands: relative to <site path> or absolute. |
|                               |                                                       | If the directory does not exist, create it.                                |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| showDebugInfo                 | auto                                                  | FE - Possible values: yes|no|auto|download. For 'auto': If a BE User is    |
|                               |                                                       | logged in, a debug information will be shown on the FE.                    |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| redirectAllMailTo             | john@doe.com                                          | If set, redirect all QFQ generated mails (Form, Report) to the specified.  |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| Database                                                                                                                                                           |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| dbInit                        | dbInit=set names utf8                                 | Global init for using the database.                                        |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| update                        | auto                                                  | | *auto*: apply DB Updates only if there is a newer version.               |
|                               |                                                       | | *always*: apply DB Updates always, especially play formEditor.sql every  |
|                               |                                                       |   time QFQ is called - *not* recommended!                                  |
|                               |                                                       | | *never*: never apply DB Updates.                                         |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| indexData                     | 1                                                     | Optional. Default: 1. Retrieve the current setting via {{_dbNameData:Y}}.  |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| indexQfq                      | 1                                                     | Optional. Default: 1. Retrieve the current setting via {{_dbNameQfq:Y}}.   |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| Security                                                                                                                                                           |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| escapeTypeDefault             | m                                                     | All variables `{{...}}` get this escape class by default.                  |
|                               |                                                       | See `variable-escape`_.                                                    |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| securityVarsHoneypot          | email,username,password                               | If empty: no check. All named variables will rendered as INPUT elements.   |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| securityAttackDelay           | 5                                                     | If an attack is detected, sleep 'x' seconds and exit PHP process.          |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| securityShowMessage           | true                                                  | If an attack is detected, show a message.                                  |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| securityGetMaxLength          | 50                                                    | GET vars longer than 'x' chars triggers an `attack-recognized`.            |
|                               |                                                       | `ExceptionMaxLength`_.                                                     |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| Form-Config                                                                                                                                                        |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| recordLockTimeoutSeconds      | 900                                                   | Timeout for record locking. After this time, a record will be replaced.    |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| enterAsSubmit                 | enterAsSubmit = 1                                     | 0: off, 1: Pressing *enter* in a form means *save* and *close*.            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| editFormPage                  | form                                                  | T3 Pagealias to edit a form.                                               |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formDataPatternError          | please check pattern error                            | Customizable error message used in validator.js. 'pattern' violation.      |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formDataRequiredError         | missing value                                         | Customizable error message used in validator.js. 'required' fields.        |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formDataMatchError            | type error                                            | Customizable error message used in validator.js. 'match' retype mismatch.  |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formDataError                 | generic error                                         | Customizable error message used in validator.js. 'no specific' given.      |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| Form-Layout                                                                                                                                                        |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| cssClassQfqContainer          | container                                             | | QFQ with own Bootstrap: 'container'.                                     |
|                               |                                                       | | QFQ already nested in Bootstrap of mainpage: <empty>.                    |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| cssClassQfqForm               | qfq-color-base                                        | Wrap around QFQ 'Form'.                                                    |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| cssClassQfqFormPill           | qfq-color-grey-1                                      | Wrap around title bar for pills: CSS Class, typically a background color.  |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| cssClassQfqFormBody           | qfq-color-grey-2                                      | Wrap around FormElements: CSS Class, typically a background color.         |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formBsColumns                 | 12                                                    | The whole form will be wrapped in 'col-md-??'. Default is 12 for 100%.     |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formBsLabelColumns            | 3                                                     | Default number of BS columns for the 'label'-column.                       |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formBsInputColumns            | 6                                                     | Default number of BS columns for the 'input'-column.                       |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formBsNoteColumns             | 3                                                     | Default number of BS columns for the 'note'-column.                        |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| extraButtonInfoInline         | <img src="info.png">                                  | Image for `extraButtonInfo`_ (inline).                                     |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| extraButtonInfoBelow          | <img src="info.png">                                  | Image for `extraButtonInfo`_ (below).                                      |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| extraButtonInfoPosition       | below                                                 | 'auto' (default) or 'below'. See `extraButtonInfo`_.                       |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| extraButtonInfoClass          | pull-right                                            | '' (default) or 'pull-right'. See `extraButtonInfo`_.                      |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| Form-Language                                                                                                                                                      |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formLanguage[ABCD]Id          | -  E.g.: 1                                            | In Typo3 configured pageLanguage id. The number after the 'L' parameter.   |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| formLanguage[ABCD]Label       | -  E.G.: english                                      | Label shown in *Form editor*, on the 'basic' tab.                          |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| saveButtonText                | -                                                     | Text on the form save button. Typically none.                              |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| saveButtonTooltip             | Save                                                  | Tooltip on the form save button.                                           |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| saveButtonClass               | btn btn-default navbar-btn                            | Bootstrap CSS class for save button on top of the form.                    |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| saveButtonClassOnChange       | alert-info btn-info                                   | Bootstrap CSS class for save button showing 'data changed'.                |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| saveButtonGlyphIcon           | glyphicon-ok                                          | Icon for the form save button.                                             |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| closeButtonText               | -                                                     | Text on the form close button. Typically none.                             |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| closeButtonTooltip            | close                                                 | Tooltip on the form close button.                                          |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| closeButtonClass              | btn btn-default navbar-btn                            | Bootstrap CSS class for close button on top of the form.                   |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| closeButtonGlyphIcon          | glyphicon-remove                                      | Icon for the form close button.                                            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| deleteButtonText              | -                                                     | Text on the form delete button. Typically none.                            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| deleteButtonTooltip           | delete                                                | Tooltip on the form delete button.                                         |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| deleteButtonClass             | btn btn-default navbar-btn                            | Bootstrap CSS class for delete button on top of the form.                  |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| deleteButtonGlyphIcon         | glyphicon-trash                                       | Icon for the form delete button.                                           |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| newButtonText                 | -                                                     | Text on the form new button. Typically none.                               |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| newButtonTooltip              | new                                                   | Tooltip on the form new button.                                            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| newButtonClass                | btn btn-default navbar-btn                            | Bootstrap CSS class for new button on top of the form.                     |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| newButtonGlyphIcon            | glyphicon-plus                                        | Icon for the form new button.                                              |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| showIdInFormTitle             | 0 (off), 1 (on)                                       | Append at the form title the current record id.                            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| cssClassColumnId              | text-muted                                            | A column in a subrecord with the name id|ID|Id gets this class.            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+


.. _config-qfq-php:

config.qfq.php
--------------

+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| Keyword                       | Example                                               | Description                                                                |
+===============================+=======================================================+============================================================================+
| DB_<n>_USER                   | DB_1_USER=qfqUser                                     | Credentials configured in MySQL                                            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| DB_<n>_PASSWORD               | DB_1_PASSWORD=1234567890                              | Credentials configured in MySQL                                            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| DB_<n>_SERVER                 | DB_1_SERVER=localhost                                 | Hostname of MySQL Server                                                   |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| DB_<n>_NAME                   | DB_1_NAME=qfq_db                                      | Database name                                                              |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| LDAP_1_RDN                    | LDAP_1_RDN='ou=Admin,ou=example,dc=com'               | Credentials for non-anonymous LDAP access. At the moment only one set of   |
| LDAP_1_PASSWORD               | LDAP_1_PASSWORD=mySecurePassword                      |                                                                            |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+
| T3_DB_NAME                    | T3_DB_NAME=specialt3dbname                            | Only necessary for inline report editing if the t3 database is not         |
|                               |                                                       | anologous to the Data db name (but ending in _t3) - see `inline-report`_.  |
+-------------------------------+-------------------------------------------------------+----------------------------------------------------------------------------+



Example: *typo3conf/config.qfq.php*: ::

    <?php

    // QFQ configuration
    //
    // Save this file as: <site path>/typo3conf/config.qfq.php

    return [
        'DB_1_USER' => '<DBUSER>',
        'DB_1_SERVER' => '<DBSERVER>',
        'DB_1_PASSWORD' => '<DBPW>',
        'DB_1_NAME' => '<DB>',

        //DB_2_USER = <DBUSER>
        //DB_2_SERVER = <DBSERVER>
        //DB_2_PASSWORD = <DBPW>
        //DB_2_NAME = <DB>

        // DB_n ...
        // ...

        // LDAP_1_RDN =
        // LDAP_1_PASSWORD =
    ];

After parsing the configuration, the following variables will be set automatically in STORE_SYSTEM:

+----------------+------------------------------------------------------------------------------------+
| _dbNameData    | Can be used to dynamically access the current selected database: {{_dbNameData:Y}} |
+----------------+------------------------------------------------------------------------------------+
| _dbNameQfq     | Can be used to dynamically access the current selected database: {{_dbNameQfq:Y}}  |
+----------------+------------------------------------------------------------------------------------+

.. _`CustomVariables`:

Custom variables
^^^^^^^^^^^^^^^^

Up to 30 custom variables can be defined in `configuration`_.

E.g. to setup a contact address and reuse the information inside your installation do: ::

   custom1: ADMINISTRATIVE_CONTACT = john@doe.com
   custom2: ADMINISTRATIVE_ADDRESS = John Doe, Hollywood Blvd. 1, L.A.
   custom3: ADMINISTRATIVE_NAME = John Doe

 * Somewhere in a `Form` or in `Report`::

      {{ADMINISTRATIVE_CONTACT:Y}}, {{ADMINISTRATIVE_ADDRESS:Y}}, {{ADMINISTRATIVE_NAME}}

It's also possible to configure such variables directly in `config.qfq.php`_.

.. _`fillStoreSystemBySql`:

Fill STORE_SYSTEM by SQL
^^^^^^^^^^^^^^^^^^^^^^^^

A specified SELECT statement in `configuration`_ in variable `fillStoreSystemBySql1` (or `2`,
or `3`) will be fired. The query should have 0 (nothing happens) or 1 row. All columns will be
**added** to the existing STORE_SYSTEM. Existing variables will be overwritten. Be careful not to overwrite system values.

This option is useful to make generic custom values, saved in the database, accessible to all QFQ Report and Forms.
Access such variables via `{{<varname>:Y}}`.

In case QFQ should stop working if a given query does not select exact one record (e.g. a missing period), define an
error message: ::

  fillStoreSystemBySql1: SELECT name FROM Person WHERE name='Doe'
  fillStoreSystemBySqlErrorMsg1: Too many or to few "Doe's" in our database

.. _`periodId`:

periodId
''''''''

This is

* a usecase, implemented via `fillStoreSystemBySql`_,
* a way to access `Period.id` with respect to the current period (the period itself is custom defined).

After a full QFQ installation:

* a table `Period` (extend / change it to your needs, fill them with your periods),
* one sample record in table `Period`,

Websites, delivering semester data, school year schedules, or any other type or periods, often need an index to the
*current* period.

In configuration_: ::

	fillStoreSystemBySql1: SELECT id AS periodId FROM Period WHERE start<=NOW() ORDER BY start DESC LIMIT 1

a variable 'periodId' will automatically computed and filled in STORE SYSTEM. Access it via `{{periodId:Y0}}`.
To get the name and current period: ::

  SELECT name, ' / ', start FROM Period WHERE id={{periodId:Y0}}

Typically, it's necessary to offer a 'previous' / 'next' link. In this example, the STORE SIP holds the new periodId: ::

  SELECT CONCAT('id={{pageId:T}}&periodId=', {{periodId:SY0}}-1, '|Next') AS _Page, ' ', name, ' ', CONCAT('id={{pageId:T}}&periodId=', {{periodId:SY0}}+1, '|Next') AS _Page FROM Period AS s WHERE s.id={{periodId:SY0}}

Take care for minimum and maximum indexes (do not render the links if out of range).

.. _`DbUserPrivileges`:

DB USER privileges
^^^^^^^^^^^^^^^^^^

The specified DB User needs privileges to the database of at least: SELECT / INSERT / UPDATE / DELETE / SHOW.

To apply automatically QFQ-'DB UPDATE' the following rights are mandatory too: CREATE / ALTER

To get access to the Typo3 installation, 'dbuser' should also have access to the Typo3 Database with at least SELECT / INSERT / UPDATE / DELETE.



.. _`ExceptionMaxLength`:

Exception for SECURITY_GET_MAX_LENGTH
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If it is necessary to use a GET variable which exceeds `securityGetMaxLength` limit, name the variable with '_<num>' at
the end. E.g. `my_long_variable_130`. Such a variable has an allowed length of 130 chars. Access the variable as
usual with the variable name: `{{my_long_variable_130:C:...}}`.


.. _local-documentation:

Local Documentation
-------------------

A HTML rendered version is available under: <your site>/typo3conf/ext/qfq/Documentation/html/Index.html

If you get a 'Page forbidden / not found' there might be some Webserver restrictions. E.g. the Typo3 example of `.htaccess`
in the Typo3 installation folder will forbid access to any extension documentation (which is a good idea on a productive
server). For a development server instead, deactivate the forbid rule of 'documentation'. In `.htaccess` (snippet from
Typo3 7.6 _.htaccess): ::

  production:   RewriteRule (?:typo3conf/ext|typo3/sysext|typo3/ext)/[^/]+/(?:Configuration|Resources/Private|Tests?|Documentation|docs?)/ - [F]
  development:  RewriteRule (?:typo3conf/ext|typo3/sysext|typo3/ext)/[^/]+/(?:Configuration|Resources/Private|Tests?|docs?)/ - [F]

.. _concept:

Concept
=======

SIPs
----

The following is a technical background information. Not needed to just use QFQ.

The SIPs (=Server Id Pairs) are uniq timestamps, created/registered on the fly for a specific browser session (=user). Every SIP is
registered on the server (= stored in a browser session) and contains one or more key/value pairs. The key/value pairs never leave
the server. The SIPs will be used:

* to protect values not to be spoofed by anyone,
* to protect values not to be altered by anyone,
* to grant access, e.g.:

  * load and save forms,
  * upload files,
  * download files,
  * PHP AJAX pages.

SIPs becomes invalid, as soon as the current browser session is destroyed. The client (= user) can't manipulate the content
of SIPs - it's only possible to reuse already registered SIPs by the user, who already owns the session.

Access privileges
-----------------

The Typo3 FE Groups can be used to implement access privileges. Such groups are assigned to

* Typo3 FE users,
* Typo3 pages,
* and/or Typo3 content records (e.g. QFQ records).

This will be used for general page structure privileges.

A `record base` privileges controlling (e.g. which user can edit
which person record) will be implicit configured, by the way that records are viewable / editable (or not) through
SQL in the specific QFQ tt-content statements.

Typo3 QFQ content element
-------------------------

Insert one or more QFQ content elements on a Typo3 page. Specify column and language per content record as wished.

The title of the QFQ content element will not be rendered on the frontend. It's only visible to the webmaster in the
backend for orientation.

QFQ Keywords (Bodytext)
^^^^^^^^^^^^^^^^^^^^^^^

 +-------------------+---------------------------------------------------------------------------------+
 | Name              | Explanation                                                                     |
 +===================+=================================================================================+
 | form              | | Formname defined in ttcontent record bodytext                                 |
 |                   | | Fix. E.g.: **form = person**                                                  |
 |                   | | - by SIP: **form = {{form:SE}}**                                              |
 |                   | | - by SQL: **form = {{SELECT c.form FROM config AS c WHERE c.id={{a:C}} }}**   |
 +-------------------+---------------------------------------------------------------------------------+
 | r                 | | <record id> The form will load the record with the specified id               |
 |                   | | - Variants: **r = 123**, by SQL: **r = {{SELECT ...}}**                       |
 |                   | | - If not specified, the default is '0'                                        |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.db        | Select a DB. Only necessary if a different than the standard DB should be used. |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.fbeg      | Start token for every field (=column)                                           |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.fend      | End token for every field (=column)                                             |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.shead     | Static start token for whole <level>, independent if records are selected       |
 |                   | Shown before `head`.                                                            |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.stail     | Static end token for whole <level>, independent if records are selected.        |
 |                   | Shown after `tail`.                                                             |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.head      | Dynamic start token for whole <level>. Only if at least one record is select.   |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.tail      | Dynamic end token for whole <level>. Only if at least one record is select.     |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.rbeg      | Start token for row.                                                            |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.rbgd      | Alternating (per row) token.                                                    |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.rend      | End token for row. Will be rendered **before** subsequent levels are processed  |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.renr      | End token for row. Will be rendered **after** subsequent levels are processed   |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.rsep      | Seperator token between rows                                                    |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.fsep      | Seperator token between fields (=columns)                                       |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.sql       | SQL Query                                                                       |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.althead   | If <level>.sql is empty, these token will be rendered.                          |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.altsql    | If <level>.sql is empty, these query will be fired. No sub queries.             |
 |                   | Shown after `althead`                                                           |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.content   | | *show* (default): content of current and sub level are directly shown.        |
 |                   | | *hide*: content of current and sub levels are stored and not shown.           |
 |                   | | *store*: content of current and sub levels are stored and shown.              |
 |                   | | To retrieve the content: `{{<level>.line.content}}`. See `syntax-of-report`_  |
 +-------------------+---------------------------------------------------------------------------------+
 | debugShowBodyText | If='1' and configuration_:*showDebugInfo: yes*, shows a tooltip with bodytext   |
 +-------------------+---------------------------------------------------------------------------------+
 | sqlLog            | Overwrites configuration_: `SQL_LOG`_ . Only affects `Report`, not `Form`.      |
 +-------------------+---------------------------------------------------------------------------------+
 | sqlLogMode        | Overwrites configuration_: `SQL_LOG_MODE`_ . Only affects `Report`, not `Form`. |
 +-------------------+---------------------------------------------------------------------------------+

.. _`qfq-database`:

QFQ Database
------------

Recommended setup for Typo3 & QFQ Installation is with *two* databases. One for the Typo3 installation and one for QFQ.
A good practice is to name both databases equal, appending the suffix '_t3' and '_db'.

When QFQ is called, it checks for QFQ system tables. If they do not exist or have a lower version than the installed qfq
version, the system tables will be automatically installed or updated.

.. _`system-tables`:

System tables
^^^^^^^^^^^^^

+---------------+------------+------------+
| Name          | Use        | Database   |
+===============+============+============+
| Clipboard     | Temporary  | QFQ        |
+---------------+------------+------------+
| Cron          | Persistent | QFQ        |
+---------------+------------+------------+
| Dirty         | Temporary  | QFQ | Data |
+---------------+------------+------------+
| Form          | Persistent | QFQ        |
+---------------+------------+------------+
| FormElement   | Persistent | QFQ        |
+---------------+------------+------------+
| FormSubmitLog | Persistent | QFQ | Data |
+---------------+------------+------------+
| MailLog       | Persistent | QFQ | Data |
+---------------+------------+------------+
| Period        | Persistent | Data       |
+---------------+------------+------------+
| Split         | Persistent | Data       |
+---------------+------------+------------+

See `Mail Log page`_ and `Form Submit Log page`_ for some Frontend views for these tables.

* Check Bug #5459 - support of system tables in different DBs not supported.

.. _`multi-database`:

Multi Database
^^^^^^^^^^^^^^

Base: T3 & QFQ
''''''''''''''

QFQ typically interacts with one database, the QFQ database. The database used by Typo3 is typically a separate one.
Theoretically it might be the same (never tested), but it's strongly recommended to use a separated QFQ database to have
no problems on Typo3 updates and to have a clean separation between Typo3 and QFQ.

QFQ: System & Data
''''''''''''''''''

QFQ itself can be separated in 'QFQ system' (see `system-tables`_) and 'QFQ data' databases (even more than one are
possible). The 'QFQ system' stores the forms, record locking, log tables and so on - `QFQ data` is for the rest.

A `Multi Database` setup is given, if 'QFQ system' is different from 'QFQ data'.

Data: Data1, Data2, ..., Data n
'''''''''''''''''''''''''''''''

Every database needs to be configured via `configuration`_ with it's own `index`.

`QFQ data` might switch between different 'data'-databases. In configuration_ one main `QFQ data` index will be specified
in `indexQfq`. If specific forms or reports should use a different database than that, `dbIndex` might change
`indexData` temporarily.

`dbIndex`: A `Report` (field `dbIndex`) as well as a `Form` (field `parameter`.`dbIndex`) can operate on a specific database.


A `Form` will:

* load the form-definition from `indexQfq` (table `Form` and `FormElement`),
* loads and save data from/in `indexData` (config.qfq.php) / `dbIndex` (form.parameter.dbIndex),
* retrieve extra information via `dbIndexExtra` - this is useful to offer information from a database and save them in a
  different one.

The simplest setup, QFQ system & data in the same database, needs no `indexQfq / indexData` definition in
configuration_ or one or both of them set to '1'

To separate QFQ system and data, indexQfq and indexData will have different indexes.


A Multi Database setup might be useful for:

* several independent Typo3 QFQ installations (each have it's own form repository) and one central database, or
* one QFQ installation which should display / load /save records from different databases, or
* a combination of the above two.

Note:

* Option 'A' is the most simple and commonly used.
* Option 'B' separate the T3 and QFQ databases on two database hosts.
* Option 'C' is like 'B' but with a shared 'QFQ data'-database between three 'Typo3 / QFQ' instances.
* Further variants are possible.

+---+----------------+--------------+-------------------------------+------------------------------+----------------------------------+
|   | Domain         | Website Host | T3                            | QFQ system                   | QFQ data                         |
+===+================+==============+===============================+==============================+==================================+
| A | standalone.edu | 'w'          | <dbHost>, <dbname>_t3, <dbnameSingle>_db                                                        |
+---+----------------+--------------+-------------------------------+------------------------------+----------------------------------+
| B | appB1.edu      | 'wApp'       | <dbHostApp>, <dbnameB1>_t3    | <dbHostB1>, <dbnameApp>_db                                      |
+---+----------------+--------------+-------------------------------+------------------------------+----------------------------------+
| B | appB2.edu      | 'wApp'       | <dbHostApp>, <dbnameB2>_t3    | <dbHostB2>, <dbnameApp>_db                                      |
+---+----------------+--------------+-------------------------------+------------------------------+----------------------------------+
| C | appC1.edu      | 'wAppC'      | <dbHostAppC>, <dbnameC1>_t3   | <dbHostC>, <dbnameSysC1>_db  | <dbHostData>_db, <dbNameData>_db |
+---+----------------+--------------+-------------------------------+------------------------------+----------------------------------+
| C | appC2.edu      | 'wAppC'      | <dbHostAppC>, <dbnameC2>_t3   | <dbHostC>, <dbnameSysC2>_db  | <dbHostData>_db, <dbNameData>_db |
+---+----------------+--------------+-------------------------------+------------------------------+----------------------------------+
| C | appC3.edu      | 'wAppC3'     | <dbHostAppC3>, <dbnameC3>_t3  | <dbHostC3>, <dbnameSysC3>_db | <dbHostData>_db, <dbNameData>_db |
+---+----------------+--------------+-------------------------------+------------------------------+----------------------------------+

In config-qfq-php_ mutliple database credentials can be prepared. Mandatory is at least one credential setup like
`DB_1_USER`, `DB_1_SERVER`, `DB_1_PASSWORD`, `DB_1_NAME`. The number '1' indicates the `dbIndex`. Increment the number
to specify further database credential setups.

Typically the credentials for `DB_1`  also have access to the T3 database.


Different QFQ versions, shared database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When using different QFQ versions and a shared 'QFQ data'-database, there is some risk of conflicting
'QFQ system' tables. Best is to always use the same QFQ version on all instances or use a Multi Database setup.

.. _debug:

Debug
=====

SQL Logging
-----------

configuration_

.. _SQL_LOG:

* *sqlLog*

  * Filename where to log SQL queries and statistical data.
  * File is relative to the `<site path>` or absolute (starting with '/').
  * Content: SQL queries and timestamp, formName/formId, fe_user, success, affected rows, newly created record
    id's and accessed from IP.
  * The global setting can be overwritten by defining `sqlLog` inside of a QFQ tt-content record.


.. _SQL_LOG_MODE:

* *sqlLogMode: all|modify|error|none*

  * *all*: logs every SQL statement.
  * *modify*: logs only statements who might potentially change data.
  * *error*: logs only queries which generate SQL errors.
  * *none*: no query logging at all.
  * The global setting can be overwritten by defining `sqlLogMode` inside of a QFQ tt-content record.

* *showDebugInfo = [yes|no|auto],[download]*

  If active, displays additional information in the Frontend (FE). This is typically helpful during development.

  * *yes*:

    * Form:

      * For every internal link/button, show tooltips with decoded SIP on mouseover.
      * Shows an 'Edit form'-button (wrench symbol) on a form. The link points to the T3 page with the :ref:`form-editor`.

    * Report: Will be configured per tt-content record.

      *debugShowBodyText = 1*

  * *no*: No debug info.

  * *auto*: Depending if there is a Typo3 BE session, set internally:

    * *showDebugInfo = yes*  (BE session exist)
    * *showDebugInfo = no*   (no BE session)

  * *download*:

    * During a download (especially by using wkhtml), temporary files are not deleted automatically. Also the
      `wkhtmltopdf` and `pdftk` commandlines will be logged to `SQL_LOG`_. Use this only to debug problems on download.

.. _REDIRECT_ALL_MAIL_TO:

Redirect all mail to (catch all)
--------------------------------

configuration_

* *redirectAllMailTo=john@doe.com*

  * During the development, it might be helpful to configure a 'catch all' email address, which QFQ uses as the final receiver
    instead of the original intended one.

  * The setting will:

    * Replace the 'To' with the configured one.
    * Clear 'CC' and 'Bcc'
    * Write a note and the original configured receiver at the top of the email body.

_`Mail Log page`

Mail Log page
-------------

For debugging purposes you may like to add a Mail Log page in the frontend.
The following QFQ code could be used for that purpose (put it in a QFQ PageContent element): ::

    # Page parameters
    1.sql = SELECT @grId := '{{grId:C0:digit}}' AS _grId
    2.sql = SELECT @summary := IF('{{summary:CE:alnumx}}' = 'true', 'true', 'false') AS _s

    # Filters
    10 {
      sql = SELECT gr.id, IF(gr.id = @grId, "' selected>", "'>"), gr.value, ' (Id: ', gr.id, ')'
               FROM gGroup AS gr
               INNER JOIN MailLog AS ml ON ml.grId = gr.id
               GROUP BY gr.id
      head = <form onchange='this.submit();' class='form-inline'><input type='hidden' name='id' value='{{pageId:T0}}'>Filter By Group: <select name='grId' class='form-control'><option value=''></option>
      rbeg = <option value='
      rend = </option>
      tail = </select>
    }
    20 {
      sql = SELECT IF(@summary = 'true', ' checked', '')
      head = <div class='checkbox'><label><input type='checkbox' name='summary' value='true'
      tail = >Summary</label></div></form>
    }

    # Mail Log
    50 {
      sql = SELECT id, '</td><td>', grId, '</td><td>', xId, '</td><td>',
                REPLACE(receiver, ',', '<br>'), '</td><td>', REPLACE(sender, ',', '<br>'), '</td><td>',
                DATE_FORMAT(modified, '%d.%m.%Y<br>%H:%i:%s'), '</td><td style="word-break:break-word;">',
                CONCAT('<b>', subject, '</b><br>', IF(@summary = 'true', CONCAT(SUBSTR(body, 1, LEAST(IF(INSTR(body, '\n') = 0, 50, INSTR(body, '\n')), IF(INSTR(body, '<br>') = 0, 50, INSTR(body, '<br>')))-1), ' ...'), CONCAT('<br>', REPLACE(body, '\n', '<br>'))) )
              FROM MailLog WHERE (grId = @grId OR @grId = 0)
              ORDER BY modified DESC
              LIMIT 100
      head = <table class="table table-condensed table-hover"><tr><th>Id</th><th>grId</th><th>xId</th><th>To</th><th>From</th><th>Date</th><th>E-Mail</th></tr>
      tail = </table>
      rbeg = <tr><td>
      rend = </td></tr>
    }

_`Form Submit Log page`

Form Submit Log page
--------------------

For debugging purposes you may like to add a Form Submit Log page in the frontend.
The following QFQ code could be used for that purpose (put it in a QFQ PageContent element): ::

    # Filters
    20.shead = <form onchange='this.submit()' class='form-inline'><input type='hidden' name='id' value='{{pageId:T0}}'>
    20 {
      sql = SELECT id, IF(id = '{{formId:SC0}}', "' selected>", "'>"), name
            FROM Form ORDER BY name
      head = <label for='formId'>Form:</label> <select name='formId' id='formId' class='form-control'><option value=0></option>
      tail = </select>
      rbeg = <option value='
      rend = </option>
    }
    30 {
      sql = SELECT feUser, IF(feUser = '{{feUser:SCE:alnumx}}', "' selected>", "'>"), feUser
            FROM FormSubmitLog GROUP BY feUser ORDER BY feUser
      head = <label for='feUser'>FE User:</label> <select name='feUser' id='feUser' class='form-control'><option value=''></option>
      tail = </select>
      rbeg = <option value='
      rend = </option>
    }
    30.stail = </form>

    # Show Log
    50 {
      sql = SELECT l.id,
          CONCAT('<b>Form</b>: ', f.name,
            '<br><b>Record Id</b>: ', l.recordId,
            '<br><b>Fe User</b>: ', l.feUser,
            '<br><b>Date</b>: ', l.created,
            '<br><b>Page Id</b>: ', l.pageId,
            '<br><b>Session Id</b>: ', l.sessionId,
            '<br><b>IP Address</b>: ', l.clientIp,
            '<br><b>User Agent</b>: ', l.userAgent,
            '<br><b>SIP Data</b>: <div style="margin-left:20px;">', "<script>var data = JSON.parse('", l.sipData,
              "'); for (var key in data) {
              document.write('<b>' + key + '</b>: ' + data[key] + '<br>'); }</script>", '</div>'),
          CONCAT("<script>var data = JSON.parse('", l.formData,
            "'); for (var key in data) {
              document.write('<b>' + key + '</b>: ' + data[key] + '<br>'); }</script>")
          FROM FormSubmitLog AS l
          LEFT JOIN Form AS f ON f.id = l.formId
          WHERE (l.formId = '{{formId:SC0}}' OR '{{formId:SC0}}' = 0)
            AND (l.feUser = '{{feUser:SCE:alnumx}}' OR '{{feUser:SCE:alnumx}}' = '')
          ORDER BY l.created DESC LIMIT 100
      head = <table class="table table-hover">
             <tr><th>Id</th><th style="min-width:250px;">Environment</th><th>Submitted Data</th>
      tail = </table>
      rbeg = <tr>
      renr = </tr>
      fbeg = <td>
      fend = </td>
    }

.. _variables:

Variable
========

Variables in QFQ are surrounded by double curly braces. Four different types of variable substitution functionality is
provided. Access to:

* `store-variables`_
* `sql-variables`_
* `row-column-variables`_
* `link-column-variables`_

Some examples, including nesting::

  # Store
  #---------------------------------------------
  {{r}}
  {{index:FS}}
  {{name:FS:alnumx:s:my default}}

  # SQL
  #---------------------------------------------
  {{SELECT name FROM person WHERE id=1234}}

  # Row columns
  #---------------------------------------------
  {{10.pId}}
  {{10.20.pId}}

  # Nesting
  #---------------------------------------------
  {{SELECT name FROM person WHERE id={{r}} }}
  {{SELECT name FROM person WHERE id={{key1:C:alnumx}} }} # explained below
  {{SELECT name FROM person WHERE id={{SELECT id FROM pf LIMIT 1}} }} # it's more efficient to use only one query

  # Link Columns
  {{p:form=Person&r=1|t:Edit Person|E|s AS link}}

Leading and trailing spaces inside curly braces are removed.

  * *{{ SELECT "Hello World"   }}* becomes *{{SELECT "Hello World"}}*
  * *{{ varname   }}* becomes *{{varname}}*

Types
-----

.. _`store-variables`:

Store variables
^^^^^^^^^^^^^^^

Syntax:  *{{VarName[:<store / prio>[:<sanitize class>[:<escape>[:<default>]]]]}}*

* Example::

  {{pId}}
  {{pId:FSE}}
  {{pId:FSE:digit}}
  {{name:FSE:alnumx:m}}
  {{name:FSE:alnumx:m:John Doe}}

* Zero or more stores might be specified to be searched for the given VarName.
* If no store is specified, the by default searched stores are: **FSRVD** (=FORM > SIP > RECORD > VARS > DEFAULT).
* If the VarName is not found in one store, the next store is searched,  up to the last specified store.
* If the VarName is not found and a default value is given, the default is returned.
* If no value is found, nothing is replaced - the string '{{<VarName>}}' remains.
* If anywhere along the line an empty string is found, this **is** a value: therefore, the search will stop.

See also:

 * `store`_
 * `variable-escape`_
 * `sanitize-class`_


.. _`sql-variables`:

SQL variables
^^^^^^^^^^^^^

* The detection of an SQL command is case *insensitive*.
* Leading  whitespace will be skipped.
* The following commands are interpreted as SQL commands:

  * SELECT
  * INSERT, UPDATE, DELETE, REPLACE, TRUNCATE
  * SHOW, DESCRIBE, EXPLAIN, SET

* A SQL Statement might contain variables, including additional SQL statements. Inner SQL queries will be executed first.
* All variables will be substituted one by one from inner to outer.
* The number of variables inside an input field or a SQL statement is not limited.

Result: string
''''''''''''''

A result of a SQL statement will be imploded over all: concat all columns of a row, concat all rows - there is no
glue string.

Result: row
'''''''''''

A few functionalities needs more than a returned string, instead separate columns are necessary. To indicate an array
result, specify those with an '!': ::

   {{!SELECT ...}}

This manual will specify the individual QFQ elements, who needs an array instead of a string. It's an error to return
a string where an array is needed and vice versa.

Database index
''''''''''''''

To access different databases in a `multi-database`_  setup, the database index can be specified after the opening curly
braces. ::

	{{[1]SELECT ... }}

For using the indexData and indexQfq (configuration_), it's a good practice to specify the variable name
instead of the numeric index. ::

   {{[{{indexData:Y}}]SELECT ...}}

If no dbIndex is given, `{{indexData:Y}}` is used.


Example
'''''''

::

  {{SELECT id, name FROM Person}}
  {{SELECT id, name, IF({{feUser:T0}}=0,'Yes','No')  FROM Person WHERE id={{r:S}} }}
  {{SELECT id, city FROM Address AS adr WHERE adr.accId={{SELECT id FROM Account AS acc WHERE acc.name={{feUser:T0}} }} }}
  {{!SELECT id, name FROM Person}}
  {{[2]SELECT id, name FROM Form}}
  {{[{{indexQfq:Y}}]SELECT id, name FROM Form}}

.. _`row-column-variables`:

Row column variables
^^^^^^^^^^^^^^^^^^^^

Syntax:  *{{<level>.<column>}}*

Only used in report to access outer columns. See `access-column-values`_ and `syntax-of-report`_.

There might be name conflicts between VarName / SQL keywords and <line identifier>. QFQ checks first for '<level>',
than for 'SQL keywords' and than for 'VarNames' in stores.

All types might be nested with each other. There is no limit of nesting variables.

Very specific: Also, it's possible that the content of a variable is again (including curly braces) a variable - this
is sometimes used in text templates, where the template is retrieved from a record and
specific locations in the text will be (automatically by QFQ) replaced by values from other sources.

General note: using this type of variables is only the second choice. First choice is `{{column:R}}` (see
`access-column-values`_) - using the STORE_RECORD is more portable cause no renumbering is needed if the level keys change.


.. _`link-column-variables`:

Link column variables
^^^^^^^^^^^^^^^^^^^^^

These variables return a link, completely rendered in HTML. The syntax and all features of `column-link`_ are available.
The following code will render a 'new person' button::

	{{p:form&form=Person|s|N|t:new person AS link}}

For better reading, the format string might be wrapped in single or double quotes (this is optional): ::

	{{"p:form&form=Person|s|N|t:new person" AS link}}

These variables are especially helpful in:

* `report`, to create create links or buttons outside of a SQL statement. E.g. in `head`, `rbeg`, ...
* `form`, to create links and buttons in labels or notes.


.. _`sanitize-class`:

Sanitize class
--------------

Values in STORE_CLIENT *C* (Client=Browser) and STORE_FORM *F* (Form, HTTP 'post') are checked against a
sanitize class. Values from other stores are not checked against any sanitize class.

* If a value violates the specific sanitize class, the value becomes `!!<name of sanitize class>!!`. E.g. `!!digit!!`.
* Variables get by default the sanitize class defined in the corresponding `FormElement`. If not defined,
  the default class is 'digit'.
* A default sanitize class can be overwritten by individual definition: *{{a:C:alnumx}}*

For QFQ variables and FormElements:

+------------------+------+-------+-----------------------------------------------------------------------------------------+
| Name             | Form | Query | Pattern                                                                                 |
+==================+======+=======+=========================================================================================+
| **alnumx**       | Form | Query | [A-Za-z][0-9]@-_.,;: /() ÀÈÌÒÙàèìòùÁÉÍÓÚÝáéíóúýÂÊÎÔÛâêîôûÃÑÕãñõÄËÏÖÜŸäëïöüÿç            |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **digit**        | Form | Query | [0-9]                                                                                   |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **numerical**    | Form | Query | [0-9.-+]                                                                                |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **allbut**       | Form | Query | All characters allowed, but not [ ]  { } % & \ #. The used regexp: '^[^\[\]{}%&\\#]+$', |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **all**          | Form | Query | no sanitizing                                                                           |
+------------------+------+-------+-----------------------------------------------------------------------------------------+


Only in FormElement:

+------------------+------+-------+-------------------------------------------------------------------------------------------+
| **auto**         | Form |       | Only supported for FormElements. Most suitable checktype is dynamically evaluated based   |
|                  |      |       | on native column definition, the FormElement type, and other info. See below for details. |
+------------------+------+-------+-------------------------------------------------------------------------------------------+
| **email**        | Form | Query | [a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}                                           |
+------------------+------+-------+-------------------------------------------------------------------------------------------+
| **pattern**      | Form |       | Compares the value against a regexp.                                                      |
+------------------+------+-------+-------------------------------------------------------------------------------------------+


Rules for CheckType Auto (by priority):

* TypeAheadSQL or TypeAheadLDAP defined: **alnumx**
* Table definition
	* integer type: **digit**
	* floating point number: **numerical**
* FE Type
	* 'password', 'note': **all**
	* 'editor', 'text' and encode = 'specialchar': **all**
* None of the above: **alnumx**


.. _`variable-escape`:

Escape
------

Variables used in SQL Statements might cause trouble by using: NUL (ASCII 0), \\n, \\r, \\, ', ", and Control-Z.

To protect the web application the following `escape` types are available:

	* 'm' - `real_escape_string() <http://php.net/manual/en/mysqli.real-escape-string.php>`_ (m = mysql)
	* 'l' - LDAP search filter values will be escaped: `ldap-escape() <http://php.net/manual/en/function.ldap-escape.php>`_ (LDAP_ESCAPE_FILTER).
	* 'L' - LDAP DN values will be escaped. `ldap-escape() <http://php.net/manual/en/function.ldap-escape.php>`_ (LDAP_ESCAPE_DN).
	* 's' - single ticks will be escaped. str_replace() of ' against \\'.
	* 'd' - double ticks will be escaped: str_replace() of " against \\".
	* 'c' - config - the escape type configured in `configuration`_.
	* ''  - the escape type configured in `configuration`_.
	* '-' - no escaping.

* The `escape` type is defined by the fourth parameter of the variable. E.g.: `{{name:FE:alnumx:m}}` (m = mysql).
* It's possible to combine different `escape` types, they will be processed in the order given. E.g. `{{name:FE:alnumx:Ls}}` (L, s).
* Escaping is typically necessary for SQL or LDAP queries.
* Be careful when escaping nested variables. Best is to escape **only** the most outer variable.
* In configuration_ a global `escapeTypeDefault` can be defined. The configured escape type applies to all substituted
  variables, who *do not* contain a *specific* escape type.
* Additionally a `defaultEscapeType` can be defined per `Form` (separate field in the *Form editor*). This overwrites the
  global definition of `configuration`. By default, every `Form.defaultEscapeType` = 'c' (=config), which means the setting
  in `configuration`_.
* To suppress a default escape type, define the `escape type` = '-' on the specific variable. E.g.: `{{name:FE:alnumx:-}}`.

Security
========

All values passed to QFQ will be:

* Checked against max. length and allowed content, on the client and on the server side. On the server side, the check
  happens before any further processing. The 'length' and 'allowed' content is specified per `FormElement`. 'digit' or
  'alnumx' is the default. Violating the rules will stop the 'save record' process (Form) or result in an empty
  value (Report). If a variable is not replaced, check the default sanitize class.

* Only elements defined in the `Form` definition or requested by `Report` will be processed.

* UTF8 normalized (normalizer::normalize) to unify different ways of composing characters. It's more a database interest,
  to work with unified data.

SQL statements are typically fired as `prepared statements` with separated variables.
Further *custom* SQL statements will be defined by the webmaster - those do not use `prepared statements` and might be
affected by SQL injection. To prevent SQL injection, every variable is by default escaped with `mysqli::real_escape_string`.

**QFQ notice**:

* Variables passed by the client (=Browser) are untrusted and use the default sanitize class 'digit' (if nothing else is
  specified). If alpha characters are submitted, the content violates `digit` and becomes therefore
  `!!<name of sanitize class>!!` - there is no error message. Best is to always use SIP (value is trustful) or at least
  digits for GET (=client) parameter (user might change those and therefore those are *not* trustful).

Get Parameter
-------------

**QFQ security restriction**:

* GET parameter might contain urlencoded content (%xx). Therefore all GET parameter will be processed by 'urldecode()'.
  As a result a text like '%nn' in GET variables will always be decoded. It's not possible to transfer '%nn' itself.

* GET values are limited to securityGetMaxLength (extension-manager-qfq-configuration_) chars - any violation will
  stop QFQ. Individual exceptions are defined via ExceptionMaxLength_.

* GET parameter 'type' and 'L' might affected by (T3, configuration dependent) cache poisoning. If they contain non digit
  values, only the first character is used (if this is a digit) or completely cleaned (else).

Post Parameter
--------------

Per `FormElement` (HTML input) the default is to `htmlspecialchars()` the input. This means &<>'" will be encoded as htmlentity
and saved as a htmlentity in the database. In case any of these characters (e.g. for HTML tags) are
required, the encoding can be disabled per FormElement: `encode=none` (default is `specialchar`).

During Form load, htmlentities are decoded again.

$_SERVER
--------

All $_SERVER vars are htmlentities encoded (all, not only specialchars!) .

Honeypot
--------

Every QFQ Form contains 'honeypot'-HTML input elements (HTML: hidden & readonly). Which of them to use is configured in
`configuration`_ (default:   'username', 'password' and 'email'). On every start of QFQ (form, report, save, ...),
these variables are tested if they are non-empty. In such a case a probably malicious bot has send the request and the
request will not be processed.

If any of the default configured variable names are needed (which will never be the case for QFQ), an explicit variable name
list have to be configured in `configuration`_.

**QFQ security restriction**:

* The honeypot variables can't be used in GET or POST as regular HTML input elements - any values of them will terminate QFQ.

Violation
---------

On any violation, QFQ will sleep `securityAttackDelaySeconds` (`configuration`_) and than exit the running PHP process.
A detected attack leads to a complete white (=empty) page.

If `securityShowMessage`: true (`configuration`_), at least a message is displayed after the delay.

Client Parameter via SIP
------------------------

Links with URL parameters, targeting to the local website, are typically SIP encoded. Instead of transferring the parameter
as part of the URL, only one unique GET parameter 's' is appended at the link. The parameter 's' is unique (equal to a
timestamp) for the user. Assigned variables are stored as a part of the PHP user session on the server.
Two users might have the same value of parameter 's', but the content is completely independent.

Variables needed by Typo3 remains on the link and are not 'sip-encoded'.

.. _`SecureDirectFileAccess`:

Secure direct file access
-------------------------

If the application uploads files, mostly it's not necessary and often a security issue, to offer a direct download of
the uploaded files. Best is to create a directory, e.g. `<site path>/fileadmin/protected` and deny direct access via
webbrowser to it. E.g. for Apache set a rule: ::

    <Directory "/var/www/html/fileadmin/protected">
        Require all denied
    </Directory>

If you only have access to `.htaccess`, create a file `<site path>/fileadmin/protected/.htaccess` with: ::

    <IfModule mod_authz_core.c>
         Require all denied
    </IfModule>

**Important**: all QFQ uploads should save files in or below such a directory.

To offer download of those files, use the reserved column name '_download' (see `download`_) or variants.

**Important**: To protect the installation against executing of uploaded malicious script code, disable PHP for the final
upload directory. E.g. `fileadmin` (Apache): ::

		<Directory "/var/www/html/fileadmin">
			php_admin_flag engine Off
		</Directory>

This is in general a good security improvement for directories with user supplied content.

File upload
-----------

By default the mime type of every uploaded file is checked against a white list of allowed mime types. The mime type of
a file can be (easily) faked by an attacker. This check is good to handle regular user file upload for specific file types
but won't help to prevent attacks against uploading and executing malicous code.

Instead prohibit the execution of user contributed files by the webserver config (`SecureDirectFileAccess`_).

Typo3 Setup - best practice
---------------------------

* Activate notification emails for every BE login (if there are only few BE users). In case the backend has been hacked,
  unusual login's (time or username) will appear: ::

        [BE][warning_email_addr] = <your email>
        [BE][warning_mode] = 1

.. _`store`:

Store
=====

Only variables that are known in a specified store can be substituted.

 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 |Name |Description                                                                             | Content                                                                        |
 +=====+========================================================================================+================================================================================+
 | B   | :ref:`STORE_BEFORE`: Record - the current record loaded in the form before any update. | All columns of the current record from the current table. See `STORE_BEFORE`_. |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | C   | :ref:`STORE_CLIENT`: POST variable, if not found: GET variable.                        | Parameter sent from the Client (=Browser). See `STORE_CLIENT`_.                |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | D   | Default values column : The *table.column* specified *default value*.                  |                                                                                |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | E   | *Empty* - always an empty string, might be helpful if a variable is empty or undefined | Any key                                                                        |
 |     | and will be used in an SQL statement.                                                  |                                                                                |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | F   | :ref:`STORE_FORM`: data not saved in database yet.                                     | All native *FormElements*. Recent values from the Browser. See: `STORE_FORM`_  |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | L   | :ref:`STORE_LDAP`: Will be filled on demand during processing of a *FormElement*.      | Custom specified list of LDAP attributes. See `STORE_LDAP`_.                   |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | M   | Column type: The *table.column* specified *type*.                                      |                                                                                |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | P   | Parent record. E.g.: on multi & copy forms the current record of the outer query.      | All columns of the MultiSQL Statement from the table for the current row       |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | R   | :ref:`STORE_RECORD`: Record - the current record loaded in the form.                   | All columns of the current record from the current table. See `STORE_RECORD`_. |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | S   | :ref:`STORE_SIP`: Client parameter 's' will indicate the current SIP, which will be    | sip, r (recordId), form. See `STORE_SIP`_.                                     |
 |     | loaded from the SESSION repo to the SIP-Store.                                         |                                                                                |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | T   | :ref:`STORE_TYPO3`: a) Bodytext (ttcontent record), b) Typo3 internal variables.       | See Typo3 tt_content record configuration. See `STORE_TYPO3`_.                 |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | U   | :ref:`STORE_USER`: per user variables, valid as long as the browser session lives.     | Set via report: '...' AS '_=<var name>' See: `STORE_USER`_,                    |
 |     |                                                                                        | `store_user_examples`_                                                         |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | V   | :ref:`STORE_VARS`: Generic variables.                                                  | See `STORE_VARS`_.                                                             |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | Y   | :ref:`STORE_SYSTEM`: a) Database, b) helper vars for logging/debugging:                |  See `STORE_SYSTEM`_.                                                          |
 |     | SYSTEM_SQL_RAW ... SYSTEM_FORM_ELEMENT_COLUMN, c) Any custom fields: CONTACT, HELP, ...|                                                                                |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+
 | 0   | *Zero* - always value: 0, might be helpful if a variable is empty or undefined and     | Any key                                                                        |
 |     | will be used in an SQL statement.                                                      |                                                                                |
 +-----+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------+

* Default *<prio>*: *FSRVD* - Form / SIP / Record / Vars / Table definition.
* Hint: Preferable, parameter should be submitted by SIP, not by Client (=URL).

  * Warning: Data submitted via 'Client' can be easily spoofed and altered.
  * Best: Data submitted via SIP never leaves the server, cannot be spoofed or altered by the user.
  * SIPs can _only_ be defined by using *Report*. Inside of *Report* use columns 'Link' (with attribute 's'), 'page?' or 'Page?'.

.. _STORE_FORM:

Store: *FORM* - F
-----------------

* Sanitized: *yes*
* Represents the values in the form, typically before saving them.
* Used for:

  * *FormElements* which will be rerendered, after a parent *FormElement* has been changed by the user.
  * *FormElement* actions, before saving the form.
  * Values will be sanitized by the class configured in corresponding the *FormElement*. By default, the sanitize class is `alnumx`.

 +---------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                            | Explanation                                                                                                                                |
 +=================================+============================================================================================================================================+
 | <FormElement name>              | Name of native *FormElement*. To get, exactly and only, the specified *FormElement* (for 'pId'): *{{pId:F}}*                               |
 +---------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+

.. _STORE_SIP:

Store: *SIP* - S
----------------

* Sanitized: *no*
* Filled automatically by creating links. E.g.:

  * in `Report` by using `_page?` or `_link` (with active 's')
  * in `Form` by using subrecords: 'new', 'edit', 'delete' links (system) or by column type `_page?`, `_link`.

 +-------------------------+-----------------------------------------------------------+
 | Name                    | Explanation                                               |
 +=========================+===========================================================+
 | sip                     | 13 char uniqid                                            |
 +-------------------------+-----------------------------------------------------------+
 | r                       | current record id                                         |
 +-------------------------+-----------------------------------------------------------+
 | form                    | current form name                                         |
 +-------------------------+-----------------------------------------------------------+
 | table                   | current table name                                        |
 +-------------------------+-----------------------------------------------------------+
 | urlparam                | all non Typo3 parameter in one string                     |
 +-------------------------+-----------------------------------------------------------+
 | <user defined>          | additional user defined link parameter                    |
 +-------------------------+-----------------------------------------------------------+

.. _STORE_RECORD:

Store: *RECORD* - R
-------------------


* Sanitized: *no*
* *Form*: Current record.
* *Report*: See `access-column-values`_
* If r=0, all values are empty.

 +------------------------+-------------------------------------------------------------------------------------------------------------------------+
 | Name                   | Type     | Explanation                                                                                                  |
 +========================+==========+==============================================================================================================+
 | <column name>          | Form     | Name of a column of the primary table (as defined in the current form). Example: *{{pId:R}}*                 |
 +------------------------+----------+--------------------------------------------------------------------------------------------------------------+
 | <column name>          | Report   | Name of a column of a previous fired SQL query. Example: *{{pId:R}}*                                         |
 +------------------------+----------+--------------------------------------------------------------------------------------------------------------+
 | &<column name>         | Report   | Name of a column of a previous fired SQL query, typically used by columns with a `special-column-names`_.    |
 |                        | (final)  | Final value. Example: '{{link:R}}' returns 'p:home&form=Person|s|b:success|t:Edit'.                          |
 |                        |          | Whereas '{{&link:R}}' returns '<span class="btn btn-success"><a href="?home&s=badcaffee1234">Edit</a></span> |
 +------------------------+----------+--------------------------------------------------------------------------------------------------------------+

.. _STORE_BEFORE:

Store: *BEFORE* - B
-------------------


* Sanitized: *no*
* Current record loaded in Form without any modification.
* If r=0, all values are empty.

This store is handy to compare new and old values of a form.

 +------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                   | Explanation                                                                                                                                      |
 +========================+==================================================================================================================================================+
 | <column name>          | Name of a column of the primary table (as defined in the current form). To get, exactly and only, the specified form *FormElement*: *{{pId:B}}*  |
 +------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------+

.. _STORE_CLIENT:

Store: *CLIENT* - C
-------------------

* Sanitized: *yes*

 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                    | Explanation                                                                                                                              |
 +=========================+==========================================================================================================================================+
 | s                       | =SIP                                                                                                                                     |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | r                       | record id. Typically stored in SIP, rarely specified on the URL                                                                          |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | HTTP_HOST               | current HTTP HOST                                                                                                                        |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | REMOTE_ADDR             | Client IP address                                                                                                                        |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | '$_SERVER[*]'           | All other variables accessible by *$_SERVER[]*. Only the often used have a pre-defined sanitize class.                                   |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | form                    | Unique name of current form                                                                                                              |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+

.. _STORE_TYPO3:

Store: *TYPO3* (Bodytext) - T
-----------------------------

* Sanitized: *no*

 +-------------------------+-------------------------------------------------------------------+----------+
 | Name                    | Explanation                                                       | Note     |
 +=========================+===================================================================+==========+
 | form                    | | Formname defined in ttcontent record bodytext                   | see note |
 |                         | | * Fix. E.g. *form = person*                                     |          |
 |                         | | * via SIP. E.g. *form = {{form:SE}}*                            |          |
 +-------------------------+-------------------------------------------------------------------+----------+
 | pageId                  | Record id of current Typo3 page                                   | see note |
 +-------------------------+-------------------------------------------------------------------+----------+
 | pageAlias               | Alias of current Typo3 page. If empty, take  pageId.              | see note |
 +-------------------------+-------------------------------------------------------------------+----------+
 | pageTitle               | Title of current Typo3 page                                       | see note |
 +-------------------------+-------------------------------------------------------------------+----------+
 | pageType                | Current selected page type (typically URL parameter 'type')       | see note |
 +-------------------------+-------------------------------------------------------------------+----------+
 | pageLanguage            | Current selected page language (typically URL parameter 'L')      | see note |
 +-------------------------+-------------------------------------------------------------------+----------+
 | ttcontentUid            | Record id of current Typo3 content element                        | see note |
 +-------------------------+-------------------------------------------------------------------+----------+
 | feUser                  | Logged in Typo3 FE User                                           |          |
 +-------------------------+-------------------------------------------------------------------+----------+
 | feUserUid               | Logged in Typo3 FE User uid                                       |          |
 +-------------------------+-------------------------------------------------------------------+----------+
 | feUserGroup             | FE groups of logged in Typo3 FE User                              |          |
 +-------------------------+-------------------------------------------------------------------+----------+
 | beUser                  | Logged in Typo3 BE User                                           |          |
 +-------------------------+-------------------------------------------------------------------+----------+
 | beUserLoggedIn          | 'yes' | 'no' - Status if a BE-User is logged in                   |          |
 +-------------------------+-------------------------------------------------------------------+----------+

* **note**: not available:

  * in :ref:`dynamic-update` or
  * by *FormElement* class 'action' with type 'beforeSave', 'afterSave', 'beforeDelete', 'afterDelete'.

.. _STORE_VARS:

Store: *VARS* - V
-----------------

* Sanitized: *no*

 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                    | Explanation                                                                                                                                |
 +=========================+============================================================================================================================================+
 | random                  | Random string with length of 32 alphanum chars (lower & upper case). This is variable is always filled.                                    |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | slaveId                 | see *FormElement* `action`                                                                                                                 |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+

.. _`store_vars_form_element_upload`:

* FormElement 'upload':

 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                    | Explanation                                                                                                                                |
 +=========================+============================================================================================================================================+
 | filename                | Original filename of an uploaded file via an 'upload'-FormElement. Valid only during processing of the current 'upload'-formElement.       |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | filenameOnly            | Like filename, but without path.                                                                                                           |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | filenameBase            | Like `filename`, but without an optional extension. E.g. filename='image.png' comes to filenameBase='image'                                |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | filenameExt             | Like `filename`, but only the optional extension. E.g. filename='image.png' comes to filenameExt='png'                                     |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | fileDestination         | Destination (path & filename) for an uploaded file. Defined in an 'upload'-FormElement.parameter. Valid: same as 'filename'.               |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | fileSize                | Size of the uploaded file.                                                                                                                 |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | mimeType                | Mimetype of the uploaded file.                                                                                                             |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+


The directive `fillStoreVar` will fill the store VARS with custom values. Existing Store VARS values will be merged together with them.
E.g.: ::

    fillStoreVar = {{!SELECT p.name, p.email FROM Person AS p WHERE p.id={{pId:S}} }}

* After filling the store, the values can be retrieved via `{{name:V}}` and `{{email:V}}`.
* Be careful by specifying general purpose variables like `id`, `r`, `pageId` and so on. This might conflict with existing variables.
* `fillStoreVar` can be used in `form-parameter`_ and `fe-parameter-attributes`_


.. _STORE_LDAP:

Store: *LDAP* - L
-----------------

* Sanitized: *yes*
* See also :ref:`LDAP`:

 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                    | Explanation                                                                                                                                |
 +=========================+============================================================================================================================================+
 | <custom defined>        | See *ldapAttributes*                                                                                                                       |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+



.. _STORE_SYSTEM:

Store: *SYSTEM* - Y
-------------------

* Sanitized: *no*

See configuration_ for a list of all settings.



.. _STORE_USER:

Store: *USER* - U
-----------------

* Sanitized: *no*

At start of a new browser session (=user calls the website the first time or was logged out before) the store is empty.
As soon as a value is set in the store, it remains as long as the browser session is alive (=until user logs out).

Values can be set via report '... AS "_=<var name>"'

See also: `store_user_examples`_

.. _LDAP:

LDAP
====

A form can retrieve data from LDAP server(s) to display or to save them. Configuration options for LDAP will be specified
in the *parameter* field of the *Form* and/or the *FormElement*. Definitions of the *FormElement* will overwrite definitions
of the *Form*. One LDAP Server can be configured per *FormElement*. Multiple *FormElements* might use individual LDAP
Server configurations.

To decide which Parameter should be placed on *Form.parameter* and which on *FormElement.parameter*: If LDAP access is ...

* only necessary in one *FormElement*, most useful setup is to specify all values in that specific *FormElement*,
* needed on multiple *FormElement*s (of the same *Form*, e.g. one *input* with *typeAhead*, one *note* and one *action*), it's more
  efficient to specify the base parameter *ldapServer*, *ldapBaseDn* in *Form.parameter* and the rest on the current
  *FormElement*.

+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| Parameter                   | Example                          | Description                                                   | Form | FormElement | Used for |
+=============================+==================================+===============================================================+======+=============+==========+
| ldapServer                  | directory.example.com            | Hostname. For LDAPS: `ldaps://directory.example.com:636`      | x    | x           | TA, FSL  |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| ldapBaseDn                  | ou=Addressbook,dc=example,dc=com | Base DN to start the search                                   | x    | x           | TA, FSL  |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| ldapAttributes              | cn, email                        | List of attributes to save in STORE_LDAP                      | x    | x           | FSL      |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| ldapSearch                  | (mail=john.doe@example.com)      | Regular LDAP search expression                                | x    | x           | FSL      |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| ldapTimeLimit               | 3 (default)                      | Maximum time to wait for an answer of the LDAP Server         | x    | x           | TA, FSL  |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| ldapUseBindCredentials      | ldapUseBindCredentials=1         | Use LDAP_1_* credentials from config-qfq-php_ for ldap_bind() | x    | x           | TA, FSL  |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLdap               | -                                | Enable LDAP as 'Typeahead' data source                        |      | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLdapSearch         | `(|(cn=*?*)(mail=*?*))`          | Regular LDAP search expression, returns upto typeAheadLimit   | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLdapSearchPrefetch | `(mail=?)`                       | Regular LDAP search expression, typically return one record   | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLdapSearchPerToken | -                                | Split search value in token and OR-combine every search with  | x    | x           | TA       |
|                             |                                  |  the individual tokens                                        |      |             |          |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLdapValuePrintf    | `'%s / %s', cn, mail`            | Custom format to display attributes, as `value`               | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLdapIdPrintf       | `'%s', mail`                     | Custom format to display attributes, as `id`                  | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLimit              | 20 (default)                     | Result will be limited to this number of entries              | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadPedantic           | typeAheadPedantic=0              | Turn off 'pedantic' mode - allow any values (see below)       | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadMinLength          | 2 (default)                      | Minimum number of characters before starting the search       | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| fillStoreLdap               | -                                | Activate `Fill STORE LDAP` with the first retrieved record    |      | x           | FSL      |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+

* *typeAheadLimit*: there might be a hard limit on the server side (e.g. 100) - which can't be extended.
* *ldapUseBindCredentials* is only necessary if `anonymous` access is not possible. RDN and password has to be configured in
  config-qfq-php_.

.. _LDAP_Typeahead:

Typeahead (TA)
--------------

*Typeahead* offers continous searching of a LDAP directoy by using a regular *FormElement* of type *text*.
The *FormElement.parameter*=*typeAheadLdap* will trigger LDAP searches on every user **keystroke**
(starting after *typeAheadMinLength* keystrokes) for the current *FormElement* - this is different from *dynamicUpdate*
(triggered by leaving focus of an input element). Typeahead delivers a list of elements.

* *FormElement.parameter.typeAheadLdap* - activate the mode *Typeahead* - no value is needed, the existence is suffucient.
* *Form.parameter* or *FormElement.parameter*:

  * *ldapServer* = `directory.example.com`
  * *ldapBaseDn* =  `ou=Addressbook,dc=example,dc=com`
  * *typeAheadLdapSearch* = `(|(cn=*?*)(mail=*?*))`
  * *typeAheadLdapValuePrintf* = `'%s / %s', cn, email`
  * *typeAheadLdapIdPrintf* = `'%s', email`
  * Optional: *ldapUseBindCredentials* = 1

All fetched LDAP values will be formatted with:

* *typeAheadLdapValuePrintf*, shown to the user in a drop-down box and
* *typeAheadLdapIdPrintf*, which represents the final data to save.

The `id/value` translation is compareable to a regular select drop-down box with id/value pairs.
Only attributes, defined in *typeAheadLdapValuePrintf* / *typeAheadLdapIdPrintf* will be fetched from the LDAP directory.
To examine all possible values of an LDAP server, use the commandline tool `ldapsearch`. E.g.::

  ldapsearch -x -h directory.example.com -L -b ou=Addressbook,dc=example,dc=com "(mail=john.doe@example.com)"

All occurrences of a '?' in *ldapSearch* will be replaced by the user data typed in via the text-*FormElement*.
The typed data will be escaped to fulfill LDAP search limitations.
Regular *Form* variables might be used on all parameter and will be evaluated during form load (!) - *not* at the time when
the user types something.

Pedantic
^^^^^^^^

The *typeAheadPedantic* mode ensures that the typed value (technically this is the value of the *id*, latest in the moment
when loosing the focus) is valid (= exists on the LDAP server or is defined in `typeAheadSql`).
If the user typed something and that is not a valid *id*, the client (=browser) will delete the input when losing the focus.
To identify the exact *id*, an additional search filter is necessary: `typeAheadLdapSearchPrefetch` - see next topic.

*typeAheadPedantic* is active by default when *typeAheadLdap* or *typeAheadSql* is defined, but can be turned off with
*typeAheadPedantic=0*.

* *Form.parameter* or *FormElement.parameter*:

  * *typeAheadPedantic=0*

Prefetch
^^^^^^^^

After 'form load' with an existing record, the user expects to see the previous saved data. In case there is an *id* to
*value* translation, the *value* does not exist in the database, instead it has to be fetched again dynamically.
A precise LDAP or SQL query has to be defined to force this:

* *Form.parameter* or *FormElement.parameter*:

  * *typeAheadLdapSearchPrefetch* = `(mail=?)`
  * *typeAheadSqlPrefetch* = `SELECT firstName, ' ', lastName FROM person WHERE id = ?`

This situation also applies in *pedantic* mode to verify the user input after each change.

PerToken
^^^^^^^^

Sometimes a LDAP server only provides attributes like 'sn' and 'givenName', but not 'displayName' or another practial
combination of multiple attributes - than it is difficult to search for 'firstname' *and* (=human AND) 'lastname'.
E.g. 'John Doe', results to search like `(|(sn=*John Doe*)(givenName=*John Doe*))` which will be probably always be empty.
Instead, the user input has to be split in token and the search string has to repeated for every token.

* *Form.parameter* or *FormElement.parameter*:

  * *typeAheadLdapSearchPerToken* - no value needed.

This will repeat the search string per token.

E.g.::

   User search string: X Y
   Ldap search string: (|(a=*?*)(b=*?*))

   Result: (& (|(a=*X*)(b=*X*)) (|(a=*Y*)(b=*Y*))

Attention: this option is only useful in specific environments. Only use it, if it is really needed. The query becomes
much more cpu / IO intensive.


.. _Fill_LDAP_STORE:

Fill STORE LDAP (FSL)
---------------------

Before processing a *FormElement*, an optional configured FSL-action loads **one** record from a LDAP directory and stores
the named attributes in STORE_LDAP. If the LDAP search query selects more than one record, only the first record is processed.
The attributes names always becomes lowercase (PHP implentation detail on get_ldap_entries()) in the store. To make
accessing STORE_LDAP easily, the keys are implemented case insensitive for this specific store. FLS is triggered during *Form*-...

* load,
* dynamic update,
* save.

The FLS happens *before* the *FormElement* processing starts. Therefore the fetched LDAP data (specified by *ldapAttributes*),
are available via `{{<attributename>:L:allbut:s}}` during the regular *FormElement* processing. Take care to specify
a sanitize class and optional escaping on further processing of those data.

Also, the STORE LDAP remains filled, during the whole form processing time. E.g. if the values are needed for a person
name and email, it's sufficient to fire one FSL on the first FormElement Action element, and use the same values during further FormElement
Action Elements.

Important: LDAP access might slow down the *Form* processing on load, update or save! The timeout (default: 3 seconds) have
to be multiplied by the number of accesses. E.g. a broken LDAP connection and 3 *FormElements* with *FSL*
results to 9 seconds delay on save. Also be prepared not to receive the expected data.

* *FormElement.parameter.fillStoreLdap* - activate the mode *Fill S* - no value is needed, the existence is sufficient.
* *Form.parameter* or *FormElement.parameter*:

  * *ldapServer* = `directory.example.com`
  * *ldapBaseDn* =  `ou=Addressbook,dc=example,dc=com`
  * *typeAheadLdapSearch* = `(|(cn=*?*)(mail=*?*))`
  * *ldapAttributes* = `givenName, sn, telephoneNumber, email`
  * *ldapSearch* = `(mail={{email:F0:alnumx:l}})`
  * Optional: *ldapUseBindCredentials* = 1

After filling the store, access the content via `{{<attributename>:L:allbut:s}}`.

Form
====

General
-------

* Forms will be created by using the *Form Editor* on the Typo3 frontend (HTML form).
* The *Form editor* itself consist of two predefined QFQ forms: *form* and *formElement* - these forms are often updated
  during the installation of new QFQ versions.
* Every form consist of a) a *Form* record and b) multiple *FormElement* records.
* A form is assigned to a  *table*. Such a table is called the *primary table* for this form.
* Forms can roughly categorized into:

  * *Simple* form: the form acts on one record, stored in one table.

    * The form will create necessary SQL commands for insert, update and delete (only primary record) automatically.

  * *Advanced* form: the form acts on multiple records, stored in more than one table.

    * Fields of the primary table acts like a *simple* form, all other fields have to be specified with *action/afterSave* records.

  * *Multi* form: the form acts simultaneously on more than one record. All records use the same *FormElements*.

    * The *FormElements* are defined as a regular *simple* / or *advanced* form, plus a SQL Query, which selects and
      iterates over all records. Those records will be loaded at the same time.

  * *Delete* form: any form can be used as a form to `delete-record`_.

* Form mode: The parameter 'r' (given via URL or via SIP) decide if the form is in mode:

  * `New`:

    * Missing parameter 'r' or 'r=0'
    * On form load, no record is displayed.
    * Saving the form creates a new primary record.
    * E.g.: `http://example.com/index.php?id=home&form=Person`  or `http://example.com/index.php?id=home&form=Person&r=0`

  * `Edit`:

    * Parameter 'r>0'
    * On form load, the specified record (<tablename>.id= <r>) will be loaded and displayed.
    * Saving the form will update the existing record.
    * E.g.: `http://example.com/index.php?id=home&form=Person&r=123`

  * Providing additional parameter:

    Often, it is necessary to store additional, for the user not visible, parameter in a record. See `form-magic`_.

* Display a form:

  * Create a QFQ tt_content record on a Typo 3 page.
  * Inside the QFQ record: `form  = <formname>`. E.g.:

    * Static: `form = Person`
    * Dynamic: `form  = {{form:SE}}`  (the left `form` is a keyword for QFQ, the right `form` is a free chooseable variable name)

  * With the `Dynamic` option, it's easily possible to use one Typo3 page and display different forms on that specific
    page. This is nice to configure few Typo 3 pages. The disadvantage is that the user might loose the navigation.


.. _record_locking:

Record locking
--------------

Forms and 'record delete'-function support basic record locking. A user opens a form: starting with the first change of content, a
record lock will be acquired in the background. If the lock is denied (another user already owns a lock on the record) an
alert is shown.
By default the record lock mode is 'exclusive' and the default timeout is 15 minutes. Both can be configured per form.
The general timeout can also be configured in configuration_ (will be copied to the form during creating the form).

The lock timeout counts from the first change, not from the last modification on the form.

If a timeout expires, the lock becomes invalid. During the next change in a form, the lock is acquired again.

A lock is assigned to a record of a table. Multiple forms, with the same primary table, uses the same lock for a given record.

If a `Form` acts on further records (e.g. via FE action), those records are not protected by this basic record locking.

If a user tries to delete a record and another user already owns a lock on that record, the delete action is denied.

If there are different locking modes in multiple forms, the most restricting mode applies for the current lock.

Exclusive
^^^^^^^^^

An existing lock on a record forbids any write action on that record.

Advisory
^^^^^^^^

An existing lock on a record informs the user that another user is currently working on that record. Nevertheless,
writing is allowed.

None
^^^^

No locking at all.

.. _comment-space-character:

Comment- and space-character
----------------------------

The following applies to the fields `Form.parameter` and `FormElement.parameter`:

* Lines will be trimmed - leading and trailing spaces will be removed.
* If a leading and/or trailing space is needed, escape it: '\\ hello world \\' > ' hello world '.
* Lines starting with a '#' are treated as a comment and will not be parsed. Such lines are treated as 'empty lines'.
* The comment sign can be escaped with '\\'.

.. _form-main:

Definition
----------

+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
| Name                    | Description                                                                                                                                        |
+=========================+====================================================================================================================================================+
|Name                     | Unique and speaking name of the *Form*. Form will be identified by this name. Use only '[a-zA-Z0-9._+-]'. _`form-name`                             |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Title                    | Title, shown on/above the form. _`form-title`                                                                                                      |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Note                     | Personal editor notes. _`form-note`                                                                                                                |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Table                    | Primary table of the form. _`form-tablename`                                                                                                       |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Primary Key              | Primary key of the indicated table. Only needed if != 'id'. _`form-primary-key`                                                                    |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Required Parameter NEW   | Name of required SIP parameter to create a new record (r=0), separated by comma. '#' as comment delimiter. See `form-requiredParameter`_           |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Required Parameter EDIT  | Name of required SIP parameter to edit an existing record (r>0), separated by comma. '#' as comment delimiter. See `form-requiredParameter`_       |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Permit New               | 'sip, logged_in, logged_out, always, never' (Default: sip): See `form-permitNewEdit`_                                                              |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Permit Edit              | 'sip, logged_in, logged_out, always, never' (Default: sip): See `form-permitNewEdit`_                                                              |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Escape Type Default      | See `variable-escape`_.                                                                                                                            |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Show button              | 'new, delete, close, save' (Default: 'new,delete,close,save'): Shown named buttons in the upper right corner of the form.  See `form-showButton`_  |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Forward Mode             | 'auto | close | no | url | url-skip-history' (Default: auto): See `form-forward`_.                                                                 |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Forward (Mode) Page      | a) URL / Typo3 page id/alias or b) Forward Mode (via '{{...}}') or combination of a) & b). See `form-forward`_.                                    |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|Parameter                |  Misc additional parameters. See `form-parameter`_.                                                                                                |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|BS Label Columns         | The bootstrap grid system is based on 12 columns. The sum of *bsLabelColumns*,                                                                     |
+-------------------------+ *bsInputColumns* and *bsNoteColumns* should be 12. These values here are the base values                                                           |
|BS Input Columns         | for all *FormElements*. Exceptions per *FormElement* can be specified per *FormElement*.                                                           |
+-------------------------+ Default: label=3, input=6, note=3. See `form-layout`_.                                                                                             |
|BS Note Columns          |                                                                                                                                                    |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|multiMode                | NOT IMPLEMENTED - 'none, horizontal, vertical' (Default 'none')                                                                                    |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|multiSql                 | NOT IMPLEMENTED - Optional. SQL Query which selects all records to edit.                                                                           |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|multiDetailForm          | NOT IMPLEMENTED - Optional. Form to open, if a record is selected to edit (double click on record line)                                            |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
|multiDetailFormParameter | NOT IMPLEMENTED - Optional. Translated Parameter submitted to detailform (like subrecord parameter)                                                |
+-------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------+

.. _`form-permitNewEdit`:

permitNew & permitEdit
^^^^^^^^^^^^^^^^^^^^^^

Depending on `r`, the following access permission will be taken:

* `r=0` - permitNew
* `r>0` - permitEdit


+------------+---------------------------------------------------------------------------------------------------+
| Option     | Description                                                                                       |
+============+===================================================================================================+
| sip        | The parameter 'form' and 'r' must be supplied via SIP or hard coded in the QFQ tt_content record. |
+------------+---------------------------------------------------------------------------------------------------+
| logged_in  | The form will only be shown if the current User is logged in as a FE User                         |
+------------+---------------------------------------------------------------------------------------------------+
| logged_out | The form will only be shown if the current User is *not* logged in as a FE User                   |
+------------+---------------------------------------------------------------------------------------------------+
| always     | No access restriction, the form will always be shown                                              |
+------------+---------------------------------------------------------------------------------------------------+
| never      | The form will never be shown                                                                      |
+------------+---------------------------------------------------------------------------------------------------+


* `sip`

  * is *always* the preferred way. With 'sip' it's not necessary to differ between logged in or not, cause the SIP
    only  exist and is only valid, if it's created via QFQ/report earlier. This means 'creating' the SIP implies
    'access granted'. The grant will be revoked when the QFQ session is destroyed - this happens when a user loggs out or
    the web browser is closed.

* `logged_in` / `logged_out`: for forms which might be displayed without a SIP, but maybe on a protected or even
  unprotected page. *The option is probably not often used.*

* `always`: such a form is always allowed to be loaded.

  * `permitNew=always`: Public accessible forms (e.g. for registration) will allow users to fill and save
    the form.

  * `permitEdit=always`: Public accessible forms will allow users to update existing data. This
    is dangerous, cause the URL parameter (recordId) 'r' might be changed by the user (URL manipulating) and therefore
    the user might see and/or change data from other users. *The option is probably not often used.*

* `never`: such a form is not allowed to be loaded.

  * `permitNew=never`: Public accessible forms. It's not possible to create new records.

  * `permitEdit=none`: Public accessible forms. It's not possible to update records.


.. _`form-requiredParameter`:

Required Parameter New|Edit
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Comma separated list of variable names. On form load, an error message will be shown in case of missing parameters.
The parameters must be given by SIP.

The list of required parameter has to be defined for `New` (r=0, create a new record) and for `Edit` (r>0, edit existing
record).

Optional a comment might be attached after the parameter definition.

E.g.: ::

  New: grId, pId # Always specify a person, grId2
  Edit: pId

.. _`form-showButton`:

showButton
^^^^^^^^^^

Display or hide the button `new`, `delete`, `close`, `save`.

* *new*: Creates a new record. If the form needs any special parameter via SIP or Client (=browser), hide this 'new' button - the necessary parameter are not provided.
* *delete*: This either deletes the current record only, or (if defined via action *FormElement* 'beforeDelete' ) any specified subrecords.
* *close*: Close the current form. If there are changes, a popup opens and ask to save / close / cancel. The last page from the history will be shown.
* *save*: Save the form.

* Default: show all buttons.

.. _`form-forward`:

Forward: Save / Close
^^^^^^^^^^^^^^^^^^^^^

Forward (=forwardMode)
''''''''''''''''''''''

After the user presses *Save*, *Close*, *Delete* or *New*, different actions are possible where the browser redirects to.

* `auto` (default) - the QFQ browser Javascript logic, decides to stay on the page or to force a redirection
  to a previous page. The decision depends on:

  * *Close* goes back (feels like close) to the previous page. Note: if there is no history, QFQ won't close the tab,
     instead a message is shown.
  * *Save* stays on the current page.

* `close` - goes back (feels like close) to the previous page. Note: if there is no history, QFQ won't close the tab,
     instead a message is shown.
* `no` - no change, the browser remains on the current side. Close does not close the page. It just triggers a save if
  there are modified data.
* `url` - the browser redirects to the URL or T3 page named in `Forward URL / Page`. Independent if the user presses `save` or `close`.
* `url-skip-history` - same as `url`, but the current location won't saved in the browser history.

Only with `Forward` == `url` | `url-skip-history`, the definition of `Forward URL / Page` becomes active.

Forward URL / Page (=forwardPage)
'''''''''''''''''''''''''''''''''

Format: [<url>] or [<mode>|<url>]

* `<url>`:

  * `http://www.example.com/index.html?a=123#bottom`
  * `website.html?a=123#bottom`
  * `?<T3 Alias pageid>&a=123#bottom, ?id=<T3 page id>&a=123#bottom`
  * `{{SELECT ...}}`
  * `<mode>|<url>`

* `<mode>` - Valid keywords are as above: `auto|close|no|url|url-skip-history`

Specifying the mode in `forwardPage` overwrites `formMode` (but only if `formMode` is `url...`).

Also regular QFQ statements like {{var}} or {{SELECT ...}} are possible in `forwardPage`. This is useful to dynamically
redirect to different targets, depending on user input or any other dependencies.

If a forwardMode 'url...' is specified and there is no `forwardPage`, QFQ falls down to `auto` mode.

On a form, the user might click 'save' or 'save,close' or 'close' (with modified data this leads to 'save,close').
The CLIENT `submit_reason` shows the user action:

* `{{submit_reason:CE:alnumx}}` = `save` or `save,close`

Example forwardPage
^^^^^^^^^^^^^^^^^^^

* `{{SELECT IF('{{formModeGlobal:S:alnumx}}'='requiredOff', 'no', 'auto') }}`
* `{{SELECT IF('{{submit_reason:CE:alnumx}}'='save', 'no', 'url'), '|http://example.com' }}`

Type: combined dynamic mode & URL/page
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Syntax: `forwardPage=<mode>|<page>`

* `forwardPage={{SELECT IF(a.url='','no','url'), '|', a.url FROM address AS a }}`


.. _form-parameter:

Parameter
^^^^^^^^^

* The following parameter are optional and can be configured in the *Form.parameter* field.

+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| Name                        | Type   | Description                                                                                              |
+=============================+========+==========================================================================================================+
| dbIndex                     | int    | Database credential index, given via `config-qfq-php`_ to let the current `Form` operate on the database.|
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| bsColumns                   | int    | Wrap the whole form in '<div class="col-md-??">                                                          |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| maxVisiblePill              | int    | Show pills upto <maxVisiblePill> as button, all further in a drop-down menu. Eg.: maxVisiblePill=3.      |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| class                       | string | HTML div with given class, surrounding the whole form. Eg.: class=container-fluid.                       |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| classTitle                  | string | CSS class inside of the `title` div. Default 'qfq-form-title'.                                           |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| classPill                   | string | HTML div with given class, surrounding the `pill` title line.                                            |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| classBody                   | string | HTML div with given class, surrounding all *FormElement*.                                                |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| extraDeleteForm             | string | Name of a form which specifies how to delete the primary record and optional slave records.              |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-pattern-error          | string | Pattern violation: Text for error message used for all FormElements of current form.                     |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-required-error         | string | Required violation: Text for error message used for all FormElements of current form.                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-match-error            | string | Match violation: Text for error message used for all FormElements of current form.                       |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-error                  | string | If none specific is defined: Text for error message used for all FormElements of current form.           |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| buttonOnChangeClass         | string | Color for save button after user modified some content or current form. E.g.: 'btn-info alert-info'.     |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapServer                  | string | FQDN Ldap Server. E.g.: directory.example.com.                                                           |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapBaseDn                  | string | E.g.: `ou=Addressbook,dc=example,dc=com`.                                                                |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapAttributes              | string | List of attributes to fill STORE_LDAP with. E.g.: cn, email.                                             |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapSearch                  | string | E.g.: `(mail={{email::alnumx:l}})`                                                                       |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapTimeLimit               | int    | Maximum time to wait for an answer of the LDAP Server.                                                   |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdap               | -      | Enable LDAP as 'Typeahead' data source.                                                                  |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdapSearch         | string | Regular LDAP search expresssion. E.g.:  `(|(cn=*?*)(mail=*?*))`                                          |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdapValuePrintf    | string | Value formatting of LDAP result, per entry. E.g.: `'%s / %s / %s', mail, roomnumber, telephonenumber`    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdapIdPrintf       | string | Key formatting of LDAP result, per entry. E.g.: `'%s', mail`                                             |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdapSearchPerToken | -      | Split search value in token and OR-combine every search with the individual tokens.                      |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLimit              | int    | Maximum number of entries. The limit is applied to the server (LDAP or SQL) and the Client.              |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadMinLength          | int    | Minimum number of characters which have to typed to start the search.                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| mode                        | string | The value `readonly` will activate a global readonly mode of the form - the user can't change any data.  |
|                             |        | See :ref:`form-mode-global`                                                                              |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| enterAsSubmit               | digit  | 0: off, 1: Pressing *enter* in a form means *save* and *close*. Takes default from configuration_.       |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| submitButtonText            | string | Show a save button at the bottom of the form, with <submitButtonText> . See `submitButtonText`_.         |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| saveButtonActive            | -      | 0: off, 1: Make the 'save'-button active on *Form* load (instead of waiting for the first user change).  |
|                             |        | The save button is still 'gray' (record not dirty), but the user can click 'save'.                       |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| saveButtonText              | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| saveButtonTooltip           | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| saveButtonClass             | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| saveButtonGlyphIcon         | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| closeButtonText             | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| closeButtonTooltip          | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| closeButtonClass            | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| closeButtonGlyphIcon        | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| deleteButtonText            | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| deleteButtonTooltip         | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| deleteButtonClass           | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| deleteButtonGlyphIcon       | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| newButtonText               | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| newButtonTooltip            | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| newButtonClass              | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| newButtonGlyphIcon          | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| extraButtonInfoClass        | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| fillStoreVar                | string | Fill the STORE_VAR with custom values. See `STORE_VARS`_.                                                |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| showIdInFormTitle           | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| formSubmitLogMode           | string | Overwrite default from configuration_                                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+

* Example:

  * maxVisiblePill = 5
  * class = container-fluid
  * classBody = qfq-form-right

.. _submitButtonText:

submitButtonText
''''''''''''''''

If specified and non empty, display a regular submit button at the bottom of the page with the given text.
This gives the form a ordinary HTML-form look'n' feel. With this option, the standard buttons on the top right border
should be hided to not confuse the user.

* Optional.
* Default: Empty

class
'''''

* Optional.
* Default: `container`
* Any CSS class name(s) can be specified.
* Check `typo3conf/ext/qfq/Resources/Public/Css/qfq-bs.css` for predefined classes.
* Typical use: adjust the floating rules of the form.

  * See: http://getbootstrap.com/css/#overview-container
  * Expand the form over the whole area: `container-fluid`

classPill
'''''''''

* Optional.
* Default: `qfq-color-grey-1`
* Any CSS class name(s) can be specified.
* Check `typo3conf/ext/qfq/Resources/Public/Css/qfq-bs.css` for predefined classes.
* Typical use: adjust the background color of the `pill title` area.
* Predefined background colors: `qfq-color-white`, `qfq-color-grey-1` (dark), `qfq-color-grey-2` (light),
  `qfq-color-blue-1` (dark), `qfq-color-blue-2`. (light)
* `classPill` is only visible on forms with container elements of type 'Pill'.

classBody
'''''''''

* Optional.
* Default: `qfq-color-grey-2`
* Any CSS class name(s) can be specified.
* Check `typo3conf/ext/qfq/Resources/Public/Css/qfq-bs.css` for predefined classes.
* Typical use:

  * adjust the background color of the *FormElement* area.
  * make all form labels right align: `qfq-form-right`.

* Predefined background colors: `qfq-color-white`, `qfq-color-grey-1` (dark), `qfq-color-grey-2` (light),
  `qfq-color-blue-1` (dark), `qfq-color-blue-2`. (light)

extraDeleteForm
'''''''''''''''

Depending on the database definition, it might be necessary to delete the primary record *and* corresponding slave records.
To not repeat such 'slave record delete definition', an 'extraDeleteForm' can be specified. If the user opens a record
in a form and clicks on the 'delete' button, a defined 'extraDeleteForm'-form will be used to delete primary and slave
records instead of using the current form.
E.g. if there are multiple different forms to work on the same table, all of theses forms might reference to the same
'extraDeleteForm'-form. This simplifies the maintenance.

The 'extraDeleteForm' parameter might be specified for a 'form' and/or for 'subrecords'

See also: `delete-record`_.

.. _form-mode-global:

Form mode global
''''''''''''''''

The Form global mode `mode` is given by default with `{{formModeGlobal:SE:alnumx}}`.

Optional it might be defined via *Form.parameter* ::

    mode=readonly|requiredOff


* `readonly`: all `FormElement` of the whole form are temporarily in `readonly` mode. This is a fast way to use an
  existing *Form* just to display the form data, without a possibility for the user to change any data of the form.

* `requiredOff`: all `FormElement` of the whole, with `mode=required`, will temporarily switch to `mode=show`. In this
  mode, the user might save the form without providing all necessary data. Later on, when application logic requires a
  final submit, this mode is not used any longer (call the form as regular without the 'formModeGlobal' parameter) and
  the form can only be saved with all data given.
  Than, e.g. an action-FormElement 'afterSave' can be used to detect the final submit situation and do some extra stuff,
  necessary for the final submit.

The following shows the same *Form* in the `regular`, `readonly` and `requiredOff` mode::

	10.sql = SELECT CONCAT('from&form=person&r=', p.id, '|Regular') as _Pagee,
	                CONCAT('from&form=person&formModeGlobal=readonly&r=', p.id, '|Readonly') as _Pagee,
	                CONCAT('from&form=person&formModeGlobal=requiredOff&r=', p.id, '|Required off') as _Pagee
	                FROM Person AS p

..

FormElements
------------

* Each *form* contains one or more *FormElement*.
* The *FormElements* are divided in three categories:

  * `class-container`_
  * `class-native`_
  * `class-action`_

* Ordering and grouping: Native *FormElements* and Container-Elements (both with feIdContainer=0) will be ordered by 'ord'.
* Inside of a container, all nested elements will be displayed.
* Technical, it's *not* necessary to configure a FormElement for the primary index column `id`.
* Additional options to a *FormElement* will be configured via the *FormElement.parameter* field (analog to *Form.parameter*
  for *Forms* ).

  * See also: `comment-space-character`_

.. _class-container:

Class: Container
----------------

* Pills are containers for 'fieldset' *and* / *or* 'native' *FormElements*.
* Fieldsets are containers for 'native' *FormElements*.
* TemplateGroups are containers for 'fieldset' *and* / *or* 'native' *FormElements*.

Type: fieldset
^^^^^^^^^^^^^^

* Native *FormElements* can be assigned to a fieldset.
* FormElement settings:

  * *name*: technical name, used as HTML identifier.
  * *label*: Shown title of the fieldset.

Type: pill (tab)
^^^^^^^^^^^^^^^^

* Pill is synonymous for a tab and looks like a tab.
* If there is at least one pill defined:

  * every native *FormElement* needs to be assigned to a pill or to a fieldset.
  * every *fieldset* needs to be assigned to a pill.

* Mode:

  * `show`: all child elements will be shown.
  * `required`: same as 'show'. This mode has no other meaning than 'show'.
  * `readonly`: technical it's like HTML/CSS `disabled`.

    * The pill title is shown, but not clickable.
    * The `FormElements` on the pill still exist, but are not reachable for the user via UI.

  * `hidden`:

    * The pill is invisible.
    * The `FormElements` on the pill still exist, but are not reachable for the user via UI.


  * Note: Independent of the *mode*, all child elements are always rendered and processed by the client/server.

* Pills are 'dynamicUpdate' aware. `title` and `mode` are optional recalculated during 'dynamicUpdate'.

* FormElement settings:

  * *name*: technical name, used as HTML identifier.
  * *label*: Label shown on the corresponding pill button or inside the drop-down menu.
  * *mode*:

    * *show*, *required*: regular mode. The pill will be shown.
    * *readonly*: the pill and it's name is visible, but not clickable.
    * *hidden*: the pill is not shown at all.

  * *modeSql*:
  * *type*: *pill*
  * *feIdContainer*: `0`  - Pill's can't be nested.
  * *tooltip*: Optional tooltip on hover. Especially helpful if the pill is set to *readonly*.
  * *parameter*:

    * *maxVisiblePill*: `<nr>` - Number of Pill-Buttons shown. Undefined means unlimited. Excess Pill buttons will be
      displayed as a drop-down menu.

Type: templateGroup
^^^^^^^^^^^^^^^^^^^

*TemplateGroups* will be used to create a series of grouped (by the given *templateGroup*) *FormElements*.

*FormElements* can be assigned to a *templateGroup*. These *templateGroup* will be rendered upto *n*-times. On 'form load'
only a single (=first) copy of the *templateGroup* will be shown. Below the last copy of the *templateGroup* an 'add'-button is
shown. If the user click on it, an additional copy of the *templateGroup* is displayed. This can be repeated up to
*templateGroup.maxLength* times. Also, the user can 'remove' previously created copies by clicking on a remove button near
beside every *templateGroup*. The first copy of a templateGroup can't be removed.

* FormElement settings:

  * *label*: Shown in the FormElement-editor *container* field.
  * *maxLength*: Maximum number of copies of the current *templateGroup*. Default: 5.
  * *bsLabelColumn*, *bsInputColumn*, *bsNoteColumn*: column widths for row with the 'Add' button.
  * *parameter*:

    * *tgAddClass*: Class of the 'add' button. Default: `btn btn-default`.
    * *tgAddText*: Text shown on the button. Default: `Add`.
    * *tgRemoveClass*: Class of the 'remove' button. Default: `btn btn-default`.
    * *tgRemoveText*: Text shown on the button. Default: `Remove`.
    * *tgClass*: Class wrapped around every copy of the *templateGroup*.
      E.g. the class `qfq-child-margin-top` adds a margin between two copies of the *templateGroup*. Default: empty

Multiple *templateGroups* per form are allowed.

The name of the native FormElements, inside the templateGroup, which represents the effective table columns, uses the placeholder
`%d`. E.g. the columns `grade1`, `grade2`, `grade3` needs a *FormElement.name* = `grade%d`. The counting will always start with 1.
The placeholder `%d` can also be used in the *FormElement.label*

Example of styling the Add/ Delete Button: :ref:`example_class_template_group`

Column: primary record
''''''''''''''''''''''

If the columns `<name>%d` are real columns on the primary table, saving and delete (=empty string) are done automatically.
E.g. if there are up to five elements `grade1, ..., grade5` and the user inputs only the first three, the remaining will be set
to an empty string.

Column: non primary record
''''''''''''''''''''''''''

Additional logic is required to load, update, insert and/or delete slave records.

Load
;;;;

On each native *FormElement* of the *templateGroup* define a SQL query in the *FormElement.value* field. The query has to
select **all** values of all possible existing copies of the *FormElement* - therefore the exclamation mark is necessary.
Also define the order. E.g. *FormElement.value*::

   {{!SELECT street FROM Address WHERE personId={{id}} ORDER BY id}}

Insert / Update / Delete
;;;;;;;;;;;;;;;;;;;;;;;;

Add an *action* record, type='afterSave', and assign the record to the given *templateGroup*.

In the parameter field define: ::

		slaveId = {{SELECT id FROM Address WHERE personId={{id}} ORDER BY id LIMIT %D,1}}
		sqlHonorFormElements = city%d, street%d
		sqlUpdate = {{UPDATE Address SET city='{{city%d:FE:alnumx:s}}', street='{{street%d:FE:alnumx:s}}'  WHERE id={{slaveId}} LIMIT 1}}
		sqlInsert = {{INSERT INTO Address (`personId`, `city`, `street`) VALUES ({{id}}, '{{city%d:FE:alnumx:s}}' , '{{street%d:FE:alnumx:s}}') }}
		sqlDelete = {{DELETE FROM Address WHERE id={{slaveId}} LIMIT 1}}

The `slaveId` needs attention: the placeholder `%d` starts always at 1. The `LIMIT` directive starts at 0 - therefore
use `%D` instead of `%d`, cause `%D` is always one below `%d` - but can **only** be used on the action element.

.. _class-native:

Class: Native
-------------

Fields:

+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
| Name                | Type                        | Description                                                                                         |
+=====================+=============================+=====================================================================================================+
|Container            | int                         | 0 or *FormElement*.id of container element (pill, fieldSet, templateGroup) part the current *Form*  |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Enabled              | enum('yes'|'no')            | Process the current FormElement                                                                     |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Dynamic Update       | enum('yes'|'no')            | In the browser, *FormElements* with "dynamicUpdate='yes'"  will be updated depending on user        |
|                     |                             | input. :ref:`dynamic-update`                                                                        |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Name                 | string                      |                                                                                                     |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Label                | string                      | Label of *FormElement*. Depending on layout model, left or on top of the *FormElement*              |
|                     |                             | Additional label description can be added by wrapping in HTML tag '<small>'                         |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Mode                 | enum('show', 'readonly',    | | *Show*: regular user input field. This is the default.                                            |
|                     | 'required',                 | | *Required*: User has to specify a value. Typically, an <empty string> represents 'no value'.      |
|                     | 'hidden' )                  | | *Readonly*: user can't change any data. Data not saved.                                           |
|                     |                             | | *Hidden*: *FormElement* is not visible.                                                           |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Mode sql             | `SELECT` statement with     | A value given here overwrites the setting from `mode`. Most useful with :ref:`dynamic-update`.      |
|                     | a value like in `mode`      | E.g.: {{SELECT IF( '{{otherFunding:FR:alnumx}}'='yes' ,'show', 'hidden' }}                          |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Class                | enum('native', 'action',    | Details below.                                                                                      |
|                     | 'container')                |                                                                                                     |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Type                 | enum('checkbox', 'date', 'time', 'datetime',  'dateJQW', 'datetimeJQW', 'extra', 'gridJQW', 'text', 'editor', 'annotate',         |
|                     | 'imageCut', 'note', 'password', 'radio', 'select', 'subrecord', 'upload', 'fieldset', 'pill', 'beforeLoad', 'beforeSave',         |
|                     | 'beforeInsert', 'beforeUpdate', 'beforeDelete', 'afterLoad', 'afterSave', 'afterInsert', 'afterUpdate', 'afterDelete',            |
|                     | 'sendMail')                                                                                                                       |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Encode               | 'none', 'specialchar'       | With 'specialchar' (default) the chars <>"'& will be encoded to their htmlentity. _field-encode     |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Check Type           | enum('auto', 'alnumx',      | See: `sanitize-class`_                                                                              |
|                     | 'digit', 'numerical',       |                                                                                                     |
|                     | 'email', 'pattern',         |                                                                                                     |
|                     | 'allbut', 'all')            |                                                                                                     |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Check Pattern        | 'regexp'                    | _`field-checkpattern`: If $checkType=='pattern': pattern to match                                   |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Order                | string                      | Display order of *FormElements* ('order' is a reserved keyword)  _`field-ord`                       |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|tabindex             | string                      |HTML tabindex attribute  _`field-tabindex`                                                           |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Size                 | string                      |Visible length of input element. Might be omitted, depending on the chosen form layout.              |
|                     |                             |Format: <width>,<height> (in characters)  _`field-size`                                              |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|BS Label Columns     | string                      | Number of bootstrap grid columns for label. By default empty, value inherits from the form.         |
|                     |                             | _`field-bsLabelColumns`                                                                             |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|BS Input Columns     | string                      | Number of bootstrap grid columns for input. By default empty, value inherits from the form.         |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|BS Note Columns      | string                      | Number of bootstrap grid columns for note. By default empty, value inherits from the form.          |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Label / Input / Note | enum(...)                   | Switch on/off opening|closing of bootstrap form classes _`field-rowLabelInputNote`                  |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Maxlength            | string                      |Maximum characters for input. _`field-maxLength`                                                     |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Note                 | string                      |Note of *FormElement*. Depending on layout model, right or below of the *FormElement*.               |
|                     |                             |Report syntax can also be used, see report-notation_                                                 |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Tooltip              | text                        |Display this text as tooltip on mouse over.  _`field-tooltip`                                        |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Placeholder          | string                      |Text, displayed inside the input element in light grey. _`field-placeholder`                         |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|value                | text                        |Default value: See field-value_                                                                      |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|sql1                 | text                        |SQL query. See individual `FormEelement`. _`sql1`                                                    |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Parameter            | text                        |Might contain misc parameter. See fe-parameter-attributes_                                           |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|feGroup              | string                      | Comma-separated list of Typo3 FE Group ID. NOT SURE IF THIS WILL BE IMPLEMENTED. Native             |
|                     |                             | *FormElements*, fieldsets and pills can be assigned to feGroups. Group status: show, hidden,        |
|                     |                             | hidden. Group Access: FE-Groups. User will be assigned to FE-Groups and the form definition         |
|                     |                             | reference such FE-groups. Easy way of granting permission.                                          |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+
|Deleted              | string                      | 'yes'|'no'.                                                                                         |
+---------------------+-----------------------------+-----------------------------------------------------------------------------------------------------+

.. _`field-value`:

FE: Value
^^^^^^^^^

By default this field is empty: QFQ will fill it with the corresponding existing column value on form load.
For a customized default value define: ::

  {{SELECT IF('{{column:RE}}'='','custom default',{{column:R}}) }}


For non primary records, this is the place to load an existing value. E.g. we're on a 'Person' detail form and would like
to edit, on the same form, a corresponding person email address (which is in a separate table): ::

  {{SELECT a.email FROM Address AS a WHERE a.pId={{id:R0}} ORDER BY a.id LIMIT 1}}

Report syntax can also be used, see report-notation_.

.. _`report-notation`:

FE: 'Report' notation
^^^^^^^^^^^^^^^^^^^^^

The FE fields 'value' and 'note' understand the `Report`_ syntax. Nested SQL queries as well as links with SIP encoding
are possible. To distinguish between 'Form' and 'Report' syntax, the first line has to be `#!report` ::

    #!report

    10.sql = SELECT ...

    20 {
      sql = SELECT ...
      5.sql = SELECT ...
    }

.. _fe-parameter-attributes:

Attributes defined in the parameter field
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

See also at specific *FormElement* definitions.

+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| Name                   | Type   | Note                                                                                                     |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-pattern-error     | string | Pattern violation: Text for error message                                                                |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-required-error    | string | Required violation: Text for error message                                                               |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-match-error       | string | Match violation: Text for error message                                                                  |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-error             | string | Violation of 'check-type': Text for error message                                                        |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| htmlBefore             | string | HTML Code wrapped before the complete *FormElement*                                                      |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| htmlAfter              | string | HTML Code wrapped after the complete *FormElement*                                                       |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| wrapRow                | string | If specified, skip default wrapping (`<div class='col-md-?'>`). Instead the given string is used.        |
+------------------------+--------+                                                                                                          |
| wrapLabel              | string |                                                                                                          |
+------------------------+--------+                                                                                                          |
| wrapInput              | string |                                                                                                          |
+------------------------+--------+                                                                                                          |
| wrapNote               | string |                                                                                                          |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| extraButtonLock        | none   | No value. Show a 'lock' on the right side of the input element. See `extraButtonLock`_                   |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| extraButtonPassword    | none   | No value. Show an 'eye' on the right side of the input element. See `extraButtonPassword`_               |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| extraButtonInfo        | string | Text. Show an 'i' on the right side of the input element. See `extraButtonInfo`_                         |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| extraButtonInfoClass   | string | By default empty. Specify any class to be assigned to wrap extraButtonInfo                               |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| autofocus              | string | See `input-option-autofocus`_                                                                            |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| checkBoxMode           | string | See `input-checkbox`_, `input-radio`_, `input-select`_                                                   |
| checked                | string |                                                                                                          |
| unchecked              | string |                                                                                                          |
| label2                 | string |                                                                                                          |
| itemList               | string |                                                                                                          |
| emptyHide              | -      |                                                                                                          |
| emptyItemAtStart       | -      |                                                                                                          |
| emptyItemAtEnd         | -      |                                                                                                          |
| buttonClass            | string |                                                                                                          |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| acceptZeroAsRequired   | string | 0|1 - Accept a '0' as a valid input. Default '0' (=0 is not a valid input)                               |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| dateFormat             | string | yyyy-mm-dd | dd.mm.yyyy                                                                                  |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| decimalFormat          | string | [precision,scale] Limits and formats input to a decimal number with the specified precision and scale.   |
|                        |        | If no precision and scale are specified, the decimal format is pulled from the table definition.         |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| showSeconds            | string | 0|1 - Shows the seconds on form load. Default: 0                                                         |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| timeIsOptional         | string | 0|1 - Used for datetime input. 0 (default): Time is required - 1: Entering a time is optional            |
|                        |        | (defaults to 00:00:00 if none entered).                                                                  |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| showZero               | string | 0|1 - Empty timestamp: '0'(default) - nothing shown, '1' - the string '0000-00-00 00:00:00' is displayed |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| retype                 | string | See `input-text`_                                                                                        |
| retypeLabel            | string |                                                                                                          |
| retypeNote             | string |                                                                                                          |
| characterCountWrap     | string |                                                                                                          |
| hideZero               | string |                                                                                                          |
| emptyMeansNull         | string |                                                                                                          |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLimit         | string | See `input-typeahead`_                                                                                   |
| typeAheadMinLength     | string |                                                                                                          |
| typeAheadSql           | string |                                                                                                          |
| typeAheadSqlPrefetch   | string |                                                                                                          |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| editor-plugins         | string | See `input-editor`_                                                                                      |
| editor-toolbar         | string |                                                                                                          |
| editor-statusbar       | string |                                                                                                          |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| form                   | string | See `subrecord-option`_                                                                                  |
| page                   | string |                                                                                                          |
| title                  | string |                                                                                                          |
| extraDeleteForm        | string |                                                                                                          |
| detail                 | string |                                                                                                          |
| subrecordTableClass    | string |                                                                                                          |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| capture                | string | See `input-upload`_                                                                                      |
| accept                 | string |                                                                                                          |
| maxFileSize            | string |                                                                                                          |
| fileDestination        | string |                                                                                                          |
| fileReplace            | string |                                                                                                          |
| autoOrient             | string |                                                                                                          |
| autoOrientCmd          | string |                                                                                                          |
| autoOrientMimeType     | string |                                                                                                          |
| chmod                  | string |                                                                                                          |
| slaveId                | string |                                                                                                          |
| sqlBefore              | string |                                                                                                          |
| sqlInsert              | string |                                                                                                          |
| sqlUpdate              | string |                                                                                                          |
| sqlDelete              | string |                                                                                                          |
| sqlAfter               | string |                                                                                                          |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| fileButtonText         | string | Overwrite default 'Choose File'                                                                          |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| fillStoreVar           | string | Fill the STORE_VAR with custom values. See `STORE_VARS`_.                                                |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| min                    | s/d/n  | Minimum and/or maximum allowed values for input field. Can be used for numbers, dates, or strings.       |
+------------------------+--------+                                                                                                          |
| max                    | s/d/n  | *Always use the international format 'yyyy-mm-dd[ hh:mm[:ss]]*                                           |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+

 * `s/d/n`: string or date or number.

Native *FormElements*
'''''''''''''''''''''
* Like 'input', 'checkbox', ...

.. _`input-option-autofocus`:

autofocus
;;;;;;;;;

The first *FormElement* with this attribute will get the focus after form load. If there is no such attribute
given to any *FormElement*, the attribute will be automatically assigned to the first editable *FormElement*.

To disable 'autofocus' on a form, set 'autofocus=0' on the first editable *FormElement*.

Note: If there are multiple pills defined on a form, only the first pill will be set with 'autofocus'.

.. _`extraButtonLock`:

extraButtonLock
;;;;;;;;;;;;;;;

* The user has to click on the lock, before it's possible to change the value. This will protect data against unwanted modification.
* After Form load, the value is shown, but not editable.
* Shows a 'lock' on the right side of an input element of type `text`, `date`, `time` or `datetime`.
* This option is not available for FormElements with `mode=readonly`.
* There is no value needed for this parameter.

.. _`extraButtonPassword`:

extraButtonPassword
;;;;;;;;;;;;;;;;;;;

* The user has to click on the eye (unhide) to see the value.
* After Form load, the data is hided by asteriks.
* Shows an 'eye' on the right side of an input element of type `text`, `date`, `time` or `datetime`.
* There is no value needed for this parameter.

.. _`extraButtonInfo`:

extraButtonInfo
;;;;;;;;;;;;;;;

* After Form load,  the `info` button/icon is shown but the information message is hidden.
* The user has to click on the `info` button/icon to see an additional message.
* The value of this parameter is the text shown to the user.
* Shows an `info` button/icon, depending of `extraButtonInfoPosition` in configuration_

  * `auto`, depending on `FormElement` type:

    * on the right side of an input element for type `text`, `date`, `time` or `datetime`,
    * below the FormElement for all other types.

  * `below`: below the FormElement for all types.

* For `FormElement` with mode `below`, a `span` element with the given class in `extraButtonInfoClass` (FE, F, configuration_)
  will be applied. E.g. this might be `pull-right` to align the `info` button/icon on the right side below the input element.

.. _`input-checkbox`:

Type: checkbox
^^^^^^^^^^^^^^

Checkboxes can be rendered in mode:

* *single*:

  * One column in a table corresponds to one checkbox.
  * The value for statuses *checked* and *unchecked* are free to choose.
  * This mode is selected, if a) *checkBoxMode* = single, or b) *checkBoxMode* is missing **and** the number of fields of the column definition is <3.
  * *FormElement.parameter*:

    * *checkBoxMode* = single (optional)
    * *checked* = <value> (optional, the value which represents 'checked')

      * If *checked* is empty or missing: If *type* = 'enum' or 'set', get first item of the definition. If *type* = string, get default.

    * *unchecked* = <value> (optional, the value which represents 'unchecked')

      * If *unchecked* is empty or missing: If *type* = 'enum' or 'set', get second item of checked. If *type* = 'string', get ''.

    * *label2* = <value>       (Text right beside checkbox) (optional)


* *multi*:

  * One column in a table represents multiple checkboxes. This is typically useful for the column type *set*.
  * The value for status *checked* are free to choose, the value for status *unchecked* is always the empty string.
  * Each field key (or the corresponding value from the key/value pair) will be rendered right beside the checkbox.
  * *FormElement.parameter*

    * *checkBoxMode* = multi
    * *itemList* - E.g.:

      * ``itemList=red,blue,orange``
      * ``itemList=1:red,2:blue,3:orange``
      * If ':' or ',' are part of key or value, it needs to escaped by '\\'.
        E.g.: `itemList=1:red\\: (with colon),2:blue\\, (with comma),3:orange``

  * *FormElement.sql1* = ``{{!SELECT id, value FROM someTable}}``
  * *FormElement.maxlength* - vertical or horizontal alignment:

     * Value: '', 0, 1 - The check boxes will be aligned vertical.
     * Value: >1 - The  check boxes will be aligned horizontal, with a linebreak every 'value' elements.

* *FormElement.parameter*:

  * *emptyHide*: Existence of this item hides an entry with an empty string. This is useful for e.g. Enums, which have an empty
    entry, but the empty value should not be selectable.
  * *emptyItemAtStart*: Existence of this item inserts an empty entry at the beginning of the selectlist.
  * *emptyItemAtEnd*: Existence of this item inserts an empty entry at the end of the selectlist.
  * *buttonClass*: Instead of the plain HTML  checkbox fields, Bootstrap
    `buttons <http://getbootstrap.com/javascript/#buttons-checkbox-radio>`_. are rendered as `checkbox` elements. Use
    one of the following `classes <http://getbootstrap.com/css/#buttons-options>`_:

    * `btn-default` (default, grey),
    * `btn-primary` (blue),
    * `btn-success` (green),
    * `btn-info` (light blue),
    * `btn-warning` (orange),
    * `btn-danger` (red).

    With a given *buttonClass*, all buttons (=radios) are rendered horizontal. A value in *FormElement.maxlength* has no effect.

* *No preselection*:

  * If a form is in 'new' mode and if there is a default value configured on a table column, such a value is shown by default.
    There might be situations, where the user should be forced to select a value (e.g. specifying the gender). An unwanted
    default value can be suppressed by specifying an explicit definition on the FormElement field `value`::

      {{<columnName>:RZ}}

    For existing records the shown value is as expected the value of the record. For new records, it's the value `0`,
    which is typically not one of the ENUM / SET values and therefore nothing is selected.


Type: date
^^^^^^^^^^

* Range datetime: '1000-01-01' to '9999-12-31' or '0000-00-00'. (http://dev.mysql.com/doc/refman/5.5/en/datetime.html)
* Optional:

  * *FormElement.parameter.dateFormat*: yyyy-mm-dd | dd.mm.yyyy

Type: datetime
^^^^^^^^^^^^^^

* Range datetime: '1000-01-01 00:00:00' to '9999-12-31 23:59:59' or '0000-00-00 00:00:00'. (http://dev.mysql.com/doc/refman/5.5/en/datetime.html)
* Optional:

  * *FormElement.parameter*:

    * *dateFormat* = yyyy-mm-dd | dd.mm.yyyy
    * *showSeconds* = 0|1 - shows the seconds. Independent if the user specifies seconds, they are displayed '1' or not '0'.
    * *showZero* = 0|1 - For an empty timestamp, With '0' nothing is displayed. With '1' the string '0000-00-00 00:00:00' is displayed.

Type: extra
^^^^^^^^^^^

* The element is not transferred to the the browser.
* The element behaves like, and can be used as, a HTML hidden input element - with the advantage that the element never
  leaves the server and therefore can't be manipulated by a user.
* The element can be used to define / precalculate values for a column, which do not already exist as a native *FormElement*.
* The element is build / computed on form load and saved alongside with the SIP parameter of the current form.
* Access the value without specifying any store (default store priority is sufficient).

.. _`input-text`:

Type: text
^^^^^^^^^^

* General input for text and number.
* *FormElement.size* =

  * <number>:  width of input element in characters. Lineheight = 1.
  * <cols>,<rows>: input element = textarea, width=<cols>, height=<rows>

* *FormElement.parameter*:

  * *retype* = 1 (optional): Current input element will be rendered twice. The form can only submitted if both elements are equal.

    * *retypeLabel* = <text> (optional): The label of the second element.
    * *retypeNote* = <text> (optional): The note of the second element.

  * *characterCountWrap* = <span class="qfq-cc-style">Count: | </span> (optional).
    Displays a character counter below the input/textarea element.
  * Also check the  fe-parameter-attributes_ *data-...-error* to customize error messages shown by the validator.
  * *hideZero* = 0|1 (optional): `with hideZero=1` a '0' in the value will be replaced by an empty string.
  * *emptyMeansNull* = [0|1] (optional): with `emptyMeansNull` or `emptyMeansNull=1` a NULL value will be written if
    the value is an empty string
  * *inputType* = number (optional). Typically the HTML tag 'type' will be 'text', 'textarea' or 'number' (detected automatically).
    If necessary, the HTML tag 'type' might be forced to a specific given value.
  * *step* = Step size of the up/down buttons which increase/decrease the number of in the input field. Optional.
    Default 1. Only useful with `inputType=number` (defined explicit via `inputType` or detected automatically).

.. _`input-typeahead`:

Type Ahead
''''''''''

Activating `typeahead` functionality offers an instant lookup of data and displaying them to the user, while the user is
typing, a drop-down box offers the results. As datasource the regular SQL connection or a LDAP query can be used.
With every keystroke (starting from the *typeAheadMinLength* characters), the already typed value will be transmitted to
the server, the lookup will be performed and the result, upto *typeAheadLimit* entries, are displayed as a drop-down box.

* *FormElement.parameter*:

  * *typeAheadLimit* = <number>. Max numbers of result records to be shown. Default is 20.
  * *typeAheadMinLength* = <number>. Minimum length to type before the first lookup starts.

Depending of the `typeahead` setup, the given FormElement will contain the displayed `value` or `id` (if an id/value dict is
configured).

Configuration via Form / FormElement
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

All of the `typeAhead*` (except `typeAheadLdap`) and `ldap*` parameter can be specified either in
*Form.parameter* or in *FormElement.parameter*.

SQL
;;;

* *FormElement.parameter*:

  * *typeAheadSql* = `SELECT ... AS 'id', ... AS 'value' WHERE name LIKE ? OR firstName LIKE ? LIMIT 100`

    * If there is only one column in the SELECT statement, that one will be used and there is no dict (key/value pair).
    * If there is no column `id` or no column `value`, than the first column becomes `id` and the second column becomes `value`.
    * The query will be fired as a 'prepared statement'.
    * The value, typed by the user, will be replaced on all places where a `?` appears.
    * All `?` will be automatically surrounded by '%'. Therefore wildcard search is implemented: `... LIKE '%<?>%' ...`

  * *typeAheadSqlPrefetch* = `SELECT firstName, ' ', lastName FROM person WHERE id = ?`

    * If the query returns several results, only the first one is returned and displayed.
    * If the query selects multiple columns, the columns are concatenated.

LDAP
;;;;

See :ref:`LDAP_Typeahead`

.. _`input-editor`:

Type: editor
^^^^^^^^^^^^

* TinyMCE (https://www.tinymce.com, community edition) is used as the QFQ Rich Text Editor.
* The content will be saved as HTML inside the database.
* All configuration and plugins will be configured via the 'parameter' field. Just prepend the word 'editor-' in front
  of each TinyMCE keyword. Check possible options under:

  * https://www.tinymce.com/docs/configure/,
  * https://www.tinymce.com/docs/plugins/,
  * https://www.tinymce.com/docs/advanced/editor-control-identifiers/#toolbarcontrols

* Bars:

  * Top: *menubar* - by default hidden.
  * Top: *toolbar* - by default visible.
  * Bottom: *statusbar* - by default hidden, exception: *min_height* and *max_height* are given via size parameter.


* The default setting in *FormElement.parameter* is::

    editor-plugins=code link searchreplace table textcolor textpattern visualchars
    editor-toolbar=code searchreplace undo redo | styleselect link table | fontselect fontsizeselect | bullist numlist outdent indent | forecolor backcolor bold italic editor-menubar=false
    editor-statusbar=false

* To deactivate the surrouding `<p>` tag, configure in *FormElement.parameter*::

     editor-forced_root_block=false

  This might have impacts on the editor. See https://www.tinymce.com/docs/configure/content-filtering/#forced_root_block

* *FormElement.size* = <min_height>,<max_height>: in pixels, including top and bottom bars. E.g.: 300,600


Type: annotate
^^^^^^^^^^^^^^

The `Formelement`.type=`annotate` is a simple grafic editor which can be used to annotate images. All modifications to
a image is saved into a JSON fabric.js data string. The current `FormElement` value is the JSON fabric.js data string.
An image, specified by `FormElement.parameter`: imageSource={{pathFileName}}, will be displayed in the background. On
form load both, the image and an optional already given JSON fabric.js data string, will be displayed. The original image
file is not modified.

* *FormElement.parameter*:

  * *imageSource* ={{pathFileName2}}     -  Background image.

By using the the `FormElement` `annotate`, the JS code `fabric.min.js` and `qfq.fabric.min.js` has to be included.
See setup-css-js_.

Type: imageCut
^^^^^^^^^^^^^^

Uploaded images can be cut or rotate via QFQ (via fabric.js). The modified image is saved under the given pathFileName.

* The 'value' of the `FormElement` has to be a valid PathFileName to an image.
* Valid image file formats are SVG, PNG, JPG, GIF.
* Invalid or missing filenames results to an empty 'imageCut' element.

* *FormElement.parameter*:

  * *resizeWidth* = <empty>|[width in pixel] - the final width of the modified image. If empty (or not given), no change.
  * *keepOriginal* = <empty>|[string] - By default: '.save'. If empty (no string given), don't keep the original. If an
    extension is given and if there is not already a <pathFileName><.extension>, than the original file is to copied to it.


Type: note
^^^^^^^^^^

An FormElement without any 'input' functionality -just to show some text. Use the typical fields 'label', 'value' and
'note' to be displayed in the corresponding three standard columns.

Type: password
^^^^^^^^^^^^^^

* Like a `text` element, but every character is shown as an asterisk.

.. _`input-radio`:

Type: radio
^^^^^^^^^^^

* Radio Buttons will be built from one of three sources:

  1. 'sql1': E.g. *{{!SELECT type AS label FROM car }}* or *{{!SELECT type AS label, typeNr AS id FROM car}}* or *{{!SHOW tables}}*.

    * Resultset format 'named': column 'label' and optional a column 'id'.
    * Resultset format 'index':

      * One column in resultset >> first column represents *label*
      * Two or more columns in resultset >> first column represents *id* and second column represents *label*.

  2. *FormElement.parameter*:

    * *itemList* = `<attribute>` E.g.: *itemList=red,blue,orange* or *itemList=1:red,2:blue,3:orange*
    * If ':' or ',' are part of key or value, it needs to escaped by '\\'.
        E.g.: `itemList=1:red\\: (with colon),2:blue\\, (with comma),3:orange`

  3. Definition of the *enum* or *set* field (only labels, ids are not possible).


* *FormElement.maxlength* = `<value>`

   * Applies only to 'plain' radio elements (not the Bootstrap 'buttonClass' from below)
   * *vertical* or *horizontal* alignment:

     * `<value>`: '', 0, 1 - The radios will be aligned *vertical*.
     * `<value>`: >1 - The readios will be aligned *horizontal*, with a linebreak every 'value' elements.


* *FormElement.parameter*:

  * *emptyHide*: Existence of this item hides an entry with an empty string. This is useful for e.g. Enums, which have an empty
    entry, but the empty value should not be selectable.
  * *emptyItemAtStart*: Existence of this item inserts an empty entry at the beginning of the selectlist.
  * *emptyItemAtEnd*: Existence of this item inserts an empty entry at the end of the selectlist.
  * *buttonClass* = <class> - Instead of the plain radio fields, Bootstrap
    `buttons <http://getbootstrap.com/javascript/#buttons-checkbox-radio>`_. are rendered as `radio` elements. Use
    one of the following `classes <http://getbootstrap.com/css/#buttons-options>`_:

    * `btn-default` (default, grey),
    * `btn-primary` (blue),
    * `btn-success` (green),
    * `btn-info` (light blue),
    * `btn-warning` (orange),
    * `btn-danger` (red).

    With a given *buttonClass*, all buttons (=radios) are rendered horizontal. A value in *FormElement.maxlength* has no effect.

* *No preselection*:

  * If there is a default configured on a table column, such a value is selected by default. If the user should actively
    choose an option, the 'preselection' can be omitted by specifying an explicit definition on the FormElement field `value`::

      {{<columnName>:RZ}}

    For existing records the shown value is as expected the value of the record. For new records, it's the value `0`,
    which is typically not one of the ENUM values and therefore nothing is selected.

.. _`input-select`:

Type: select
^^^^^^^^^^^^

* Select lists will be built from one of three sources:

  * *FormElement.sql1* = `{{!<SQL Query}}`

    * E.g. *{{!SELECT type AS label FROM car }}* or *{{!SELECT type AS label, typeNr AS id FROM car}}* or *{{!SHOW tables}}*.

    * Resultset format 'named': column 'label' and optional a column 'id'.
    * Resultset format 'index':

      * One column in resultset >> first column represents *label*
      * Two or more columns in resultset >> first column represents *id* and second column represents *label*.

  * *FormElement.parameter*:

    * *itemList* = `<attribute>` - E.g.: *itemList=red,blue,orange* or *itemList=1:red,2:blue:3:orange*
    * If ':' or ',' are part of key or value, it needs to escaped by '\\'.
        E.g.: `itemList=1:red\\: (with colon),2:blue\\, (with comma),3:orange`

  * Definition of the *enum* or *set* field (only labels, ids are not possible).

* *FormElement.size* = `<value>`

  * `<value>`: <empty>|0|1: drop-down list.
  * `<value>`: >1: Select field with *size* rows height. Multiple selection of items is possible.

* *FormElement.parameter*:

  * *emptyItemAtStart*: Existence of this item inserts an empty entry at the beginning of the selectlist.
  * *emptyItemAtEnd*: Existence of this item inserts an empty entry at the end of the selectlist.
  * *emptyHide*: Existence of this item hides the empty entry. This is useful for e.g. Enums, which have an empty
    entry and the empty value should not be an option to be selected.

.. _`subrecord-option`:

Type: subrecord
^^^^^^^^^^^^^^^

The *FormElement* type 'subrecord' renders a list of records (so called secondary records), typically to show, edit, delete
or add new records. The list is defined as a SQL query. The number of records shown is not limited. These *FormElement*
will be rendered inside the form as a HTML table.

* *mode / modeSql* = `<type/value>`

  * *show / required*: the regular mode to show the subrecords
  * *readonly*: New / Edit / Delete Buttons are disabled
  * *hidden*: The FormElement is rendered, but hidden with `display='none'`.

* *dynamicUpdate* - not supported at the moment.

* *sql1* = `{{!<SQL Query}}`

   * SQL query to select records. E.g.::

      {{!SELECT addr.id AS id, CONCAT(addr.street, addr.streetnumber) AS a, addr.city AS b, addr.zip AS c FROM Address AS addr}}

  * Notice the **exclamation mark** after '{{' - this is necessary to return an array of elements, instead of a single string.
  * Exactly one column **'id'** has to exist; it specifies the primary record for the target form.
    In case the id should not be visible to the user, it has to be named **'_id'**.
  * Column name: *[title=]<title>[|[maxLength=]<number>][|nostrip][|icon][|link][|url][|mailto][|_rowClass][|_rowTooltip]*

    * If the keyword is used, all parameter are position independent.
    * Parameter are separated by '|'.
    * *[title=]<text>*: Title of the column. The keyword 'title=' is optional. Columns with a title starting with '_' won't be rendered.
    * *[maxLength=]<number>*: Max. number of characters displayed per cell. The keyword 'maxLength=' is optional. Default
       maxLength '20'. A value of '0' means no limit. This setting also affects the title of the column.
    * *nostrip*: by default, html tags will be stripped off the cell content before rendering. This protects the table
      layout. 'nostrip' deactivates the cleaning to make pure html possible.
    * *icon*: the cell value contains the name of an icon in *typo3conf/ext/qfq/Resources/Public/icons*. Empty cell values
      will omit an html image tag (=nothing rendered in the cell).
    * *link*: value will be rendered as described under :ref:`column-link`
    * *url*: value will be rendered as a href url.
    * *mailto*: value will be rendered as a href mailto.
    * *_rowClass*

      * The value is a CSS class name(s) which will be rendered in the *<tr class="<_rowClass>">* of the subrecord table.
      * The column itself is not rendered.
      * By using Bootstrap, the following predefined classes are available:

        * Text color: *text-muted|text-primary|text-success|text-info|text-warning|text-danger* (http://getbootstrap.com/css/#helper-classes)
        * Row background: *active|success|info|warning|danger* (http://getbootstrap.com/css/#tables-contextual-classes)

    * *_rowTooltip*

      * Defines the title attribute (=tooltip) of a subrecord table row.

    * Examples::

         {{!SELECT id, note1 AS 'Comment', note2 AS 'Comment|50' , note3 AS 'title=Comment|maxLength=100|nostrip', note4 AS '50|Comment',
         'checked.png' AS 'Status|icon', email AS 'mailto', CONCAT(homepage, '|Homepage') AS 'url',
         CONCAT('d|s|F:', pathFileName) AS 'Download|link',
         ELT(status,'info','warning','danger') AS '_rowClass', help AS '_rowTooltip' ...}}

* *FormElement.parameter*

  * *form* = `<form name>` - Target form, e.g. *form=person*
  * *page* = `<T3 page alias or id>` - Target page with detail form. If none specified, use the current page.
  * *extraDeleteForm*: Optional. The per row delete Button will reference the form specified here (for deleting) instead of the default (*form*).
  * *detail* = `<string>` - Mapping of values from

    * a) the primary form,
    * b) the current row,
    * c) any constant or '{{...}}' -

    to the target form (defined via `form=...`).

    * Syntax::

        <source table column name 1|&constant 1>:<target column name 1>[,<source table column name 2|&constant 2>:<target column name 2>][...]

    * Example: *detail=id:personId,rowId:secId,&12:xId,&{{a}}:personId*  (rowId is a column of the current selected row defined by sql1)
    * By default, the given value will overwrite values on the target record. In most situations, this is the wished behaviour.
    * Exceptions of the default behaviour have to be defined on the target form in the corresponding *FormElement* in the
      field *value* by changing the default Store priority definition. E.g. `{{<columnName>:RS0}}` - For existing records,
      the store `R` will provide a value. For new records, store `R` is empty and store S will be searched for a value:
      the value defined in `detail` will be choosen. At last the store '0' is defined as a fallback.
    * *source table column name*: E.g. A person form is opened with person.id=5 (r=5). The definition `detail=id:personId`
      and `form=address` maps person.id to address.personId. On the target record, the column personId becomes '5'.
    * *Constant '&'*: Indicate a 'constant' value. E.g. `&12:xId` or `{{...}}` (all possibilities, incl. further SELECT
      statements) might be used.

  * *subrecordTableClass*: Optional. Default: 'table table-hover qfq-subrecord-table'. If given, the default will be overwritten.
    This parameter is helpful if you want to add tablesorting to your subrecord - example: ::

	  subrecordTableClass = table table-hover qfq-subrecord-table tablesorter tablesorter-pager

  * *subrecordColumnTitleEdit*: Optional. Will be rendered as the column title for the new/edit column.
  * *subrecordColumnTitleDelete*: Optional. Will be rendered as the column title for the delete column.

**Subrecord DragAndDrop**

Subrecords inherently support drag-and-drop, see also `drag_and_drop`_.
The following parameters can be used in the `parameter` field to customize/activate drag-and-drop:

* *orderInterval*: The order interval to be used, default is 10.
* *dndTable*: The table that contains the records to be ordered.
  If not given, the table name of the form specified via `form=...` is used.
* *orderColumn*: The dedicated order column in the specified dndTable (needs to match a column in the table definition).
  Default is `ord`.

If `dndTable` is an actual table with a column `orderColumn`, QFQ automatically applies drag-and-drop logic
to the rendered subrecord. It does so by using the subrecord field *sql1*. The `sql1` query should
therefore include both a column id (or _id) and ord (or _ord).

Tips:

* If you want to deactivate a drag-and-drop that QFQ automatically renders, set the `orderColumn` to a non-existing column.
  E.g., `orderColumn = nonExistingColumn`. This will deactivate drag-and-drop.
* In order to evaluate the `sql1` query dynamically during a drag-and-drop event, the fill store R (with the current form)
  is loaded. Currently, SIP parameters and other variables are not supplied to be evaluated during a drag-and-drop event.
  This may be added later upon request.
* If the subrecord is rendered with drag-and-drop active, but the order is not affected upon reload, there is
  most likely a problem with evaluating the `sql1` query at runtime.

Type: time
^^^^^^^^^^

* Range time: '00:00:00' to '23:59:59' or '00:00:00'. (http://dev.mysql.com/doc/refman/5.5/en/datetime.html)
* Optional:
* *FormElement.parameter*

  * *showSeconds* = `0|1` - shows the seconds. Independent if the user specifies seconds, they are displayed '1' or not '0'.
  * *showZero* =  `0|1` - For an empty timestamp, With '0' nothing is displayed. With '1' the string '00:00[:00]' is displayed.

.. _`input-upload`:

Type: upload
^^^^^^^^^^^^

An upload element is based on a 'file browse'-button and a 'trash'-button (=delete). Only one of them is shown at a time.
The 'file browse'-button is displayed, if there is no file uploaded already.
The 'trash'-button is displayed, if there is a file uploaded already.

After clicking on the browse button, the user select a file from the local filesystem.
After choosing the file, the upload starts immediately, shown by a turning wheel. When the server received the whole file
and accepts (see below) the file, the 'file browse'-button disappears and the filename is shown, followed by a 'trash'-button.
Either the user is satisfied now or the user can delete the uploaded file (and maybe upload another one).

Until this point, the file is cached on the server but not copied to the `fileDestination`. The user have to save the
current record, either to finalize the upload and/or to delete a previously uploaded file.

The FormElement behaves like a

* 'native FormElement' (showing controls/text on the form) as well as an
* 'action FormElement' by firing queries and doing some additional actions during form save.

Inside the *Form editor* it's shown as a 'native FormElement'.
During saving the current record, it behaves like an action FormElement
and will be processed after saving the primary record and before any action FormElements are processed.

* *FormElement.value* = `<string>` - By default, the full path of any already uploaded file is shown. To show something
  different, e.g. only the filename, define: ::

	 a) {{filenameBase:V}}
	 b) {{SELECT SUBSTRING_INDEX( '{{pathFileName:R}}', '/', -1)  }}

See also `downloadButton`_ to offer a download of an uploaded file.

* *FormElement.parameter*:

  * *capture* = `camera` - On a smartphone, after pressing the 'open file' button, the camera will be opened and a
    choosen picture will be uploaded. Automatically set/overwrite `accept=image/*`.

  * *accept* = `<mime type>,image/*,video/*,audio/*,.doc,.docx,.pdf`

    * List of mime types (also known as 'media types'): http://www.iana.org/assignments/media-types/media-types.xhtml
    * If none is specified, 'application/pdf' is set. This forces that always (!) one type is specified.
    * One or more media types might be specified, separated by ','.
    * Different browser respect the given definitions in different ways. Typically the 'file choose' dialog offer:

      * the specified mime type (some browers only show 'custom', if more than one mime type is given),
      * the option 'All files' (the user is always free to **try** to upload other filetypes) - but the server won't accept them,
      * the 'file choose' dialog only offers files of the selected (in the dialog) type.

    * If for a specific filetype is no mime type available, the definition of file extension(s) is possible. This is **less
      secure**, cause there is no *content* check on the server after the upload.

  * *maxFileSize* = `<size>` - max filesize in bytes (no unit), kilobytes (k/K) or megabytes (m/M) for an uploaded file. Default: 10M.

  * *fileDestination* = `<pathFileName>` - Destination where to copy the file. A good practice is to specify a relative `fileDestination` -
    such an installation (filesystem and database) are moveable.

    * If the original filename should be part of `fileDestination`, the variable *{{filename}}* (STORE_VARS) can be used. Example ::

        fileDestination={{SELECT 'fileadmin/user/pictures/', p.name, '-{{filename}}' FROM Person AS p WHERE p.id={{id:R0}} }}

      * Several more variants of the filename and also mimetype and filesize are available. See `store_vars_form_element_upload`_.

      * The original filename will be sanitized: only '<alnum>', '.' and '_' characters are allowed. German 'umlaut' will
        be replaced by 'ae', 'ue', 'oe'. All non valid characters will be replaced by '_'.

    * If a file already exist under `fileDestination`, an error message is shown and 'save' is aborted. The user has no
      possibility to overwrite the already existing file. If the whole workflow is correct, this situation should no
      arise. Check also *fileReplace* below.

    * All necessary subdirectories in `fileDestination` are automatically created.

    * Using the current record id in the `fileDestination`: Using {{r}} is problematic for a 'new' primary record: that
      one is still '0' at the time of saving. Use `{{id:R0}}` instead.

    * Uploading of malicious code (e.g. PHP files) is hard to detect. The default mime type check can be easily faked
      by an attacker. Therefore it's recommended to use a `fileDestination`-directory, which is secured against script
      execution (even if the file has been uploaded, the webserver won't execute it) - see `SecureDirectFileAccess`_.

  * *sqlBefore*,  *sqlAfter*: available in :ref:`Upload simple mode`  and :ref:`Upload advanced mode`.
  * *slaveId*,  *sqlInsert*, *sqlUpdate*, *sqlDelete*, *sqlUpdate*: available only in :ref:`Upload advanced mode`.

  * `fileSize` / `mimeType`

    * In :ref:`Upload simple mode` the information of `fileSize` and `mimeType` will be automatically updated on the current
      record, if table columns `fileSize` and/or `mimeType` exist.

      * If there are more than one Upload FormElement in a form, the automatically update for `fileSize` and/or `mimeType`
        are not done automatically.

    * In :ref:`Upload advanced mode`  the `fileSize` and / or `mimeType`  have to be updated with an explicit SQL statement::

        sqlAfter = {{UPDATE Data SET mimeType='{{mimeType:V}}', fileSize={{fileSize:V}} WHERE id={{id:R}} }}

  * *fileReplace* = `always` - If `fileDestination` exist - replace it by the new one.

  * *chmod* = <unix file permission mode> - e.g. `660` for owner and group read and writeable. Only the numeric mode is allowed.

  * autoOrient: images might contain EXIF data (e.g. captured via mobile phones) incl. an orientation tag like TopLeft,
    BottomRight and so on. Web-Browser and other grafic programs often understand and respect those information and rotate
    such images automatically. If not, the image might be displayed in an unwanted oritentation.
    With active option 'autoOrient', QFQ tries to normalize such images via 'convert' (part of ImageMagick). Especially
    if images are processed by the QFQ internal 'Fabric'-JS it's recommended to normalize images first. The normalization
    process does not solve all orientation problems.

      * *autoOrient* = [0|1]
      * *autoOrientCmd* = 'convert -auto-orient {{fileDestination:V}} {{fileDestination:V}}.new; mv {{fileDestination:V}}.new {{fileDestination:V}}'
      * *autoOrientMimeType* = image/jpeg,image/png,image/tiff

    If the defaults for `autoOrientCmd` and `autoOrientMimeType` are sufficient, it's not necessary to specify them.

.. _`downloadButton`:

  * *downloadButton* = `t:<string>` - If given, shows a button to download the previous uploaded file - instead of the string given in
    `fe.value`. The button is only shown if `fe.value` points to a readable file on the server.

    * If `downloadButton` is empty, just shows the regular download glyph.
    * To just show the filename: `downloadButton = t:{{filenameOnly:V}}`
    * Additional attributes might be given like `downloadButton = t:Download|o:check file`. Please check `download`_.

      * The following attributes are hard coded (can't be changed): `s|M:file|d|F`

  * fileSplit, fileDestinationSplit, tableNameSplit: see split-pdf-upload_

  * Excel Import: QFQ offers functionality to directly import excel data into the database. This functionality can
    optionally be combined with saving the file by using the above parameters like `fileDestination`.

    * *importToTable*: <mariadb.tablename> - **Required**. Providing this parameter activates the import. If the table
      doesn't exist, it will be created.
    * *importToColumn*: <col1>,<col2>,... - If none provided, the Excel column names A, B, ... are used. Note: These
      have to match the table's column names if the table already exists.
    * *importRegion*: [tab],[startColumn],[startRow],[endColumn],[endRow]|... - All parts are optional (default:
      entire 1st sheet). Tab can either be given as an index (1-based) or a name. start/endColumn can be given either
      numerically (1, 2, ...) or by column name (A, B, ...). Note that you can specify several regions to import.
    * *importMode*: `append` (default) | `replace` - The data is either appended or replace in the specified table.
    * *importType*: `auto` (default) | `xls` | `xlsx` | `ods` | `csv` - Define what kind of data should be expected by the
      Spreadsheet Reader. `auto` should work fine in most cases.


Immediately after the upload finished (before the user press save), the file will be checked on the server for it's
content or file extension (see 'accept').

The maximum size is defined by the minimum of `upload_max_filesize`, `post_max_size` and `memory_limit` (PHP script) in the php.ini.

In case of broken uploads, please also check `max_input_time` in php.ini.

Deleting a record and the referenced file
'''''''''''''''''''''''''''''''''''''''''

If the user deletes a record (e.g. pressing the delete button on a form) which contains reference(s) to files, such files
are deleted too. Slave records, which might be also deleted through a 'delete'-form, are *not* checked for file references
and therefore such files are not deleted on the filesystem.

Only column(name)s which contains `pathFileName` as part of their name, are checked for file references.

If there are other records, which references the same file, such files are not deleted.
It's a very basic check: just the current column of the current table is compared. In general it's not a good idea to
have multiple references to a single file. Therefore this check is just a fallback.

.. _Upload simple mode:

Upload simple mode
;;;;;;;;;;;;;;;;;;

Requires: *'upload'-FormElement.name = 'column name'* of an column in the primary table.

After moving the file to `fileDestination`, the current record/column will be updated to `fileDestination`.
The database definition of the named column has to be a string variant (varchar, text but not numeric or else).
On form load, the column value will be displayed as the whole value (pathFileName)

Deleting an uploaded file in the form (by clicking on the trash near beside) will delete
the file on the filesystem as well. The column will be updated to an empty string.

This happens automatically without any further definiton in the 'upload'-FormElement.

Multiple 'upload'-FormElements per form are possible. Each of it needs an own table column.

.. _Upload advanced mode:

Upload advanced mode
;;;;;;;;;;;;;;;;;;;;

Requires: *'upload'-FormElement.name* is unknown as a column in the primary table.

This mode will serve further database structure scenarios.

A typical name for such an 'upload'-FormElement, to show that the name does not exist in the table, might start with 'my', e.g. 'myUpload1'.

* *FormElement.value* = `<string>` - The path/filename, shown during 'form load' to indicate a previous uploaded file, has to be queried
  with this field. E.g.::

      {{SELECT pathFileNamePicture FROM Note WHERE id={{slaveId}} }}

* *FormElement.parameter*:

  * *fileDestination* = `<pathFileName>` - determine the path/filename. E.g.::

     fileDestination=fileadmin/person/{{name:R0}}_{{id:R}}/uploads/picture_{{filename}}

  * *slaveId* = `<id>` - Defines the target record where to retrieve and store the path/filename of the uploaded file. Check also :ref:`slave-id`. E.g.::

      slaveId={{SELECT id FROM Note WHERE pId={{id:R0}} AND type='picture' LIMIT 1}}

  * *sqlBefore* = `{{<query>}}` - fired during a form save, before the following queries are fired.

  * *sqlInsert* = `{{<query>}}` - fired if `slaveId=0` and an upload exist (user has choosen a file)::

      sqlInsert={{INSERT INTO Note (pId, type, pathFileName) VALUE ({{id:R0}}, 'image', '{{fileDestination}}') }}

  * *sqlUpdate* = `{{<query>}}` - fired if `slaveId>0` and an upload exist (user has choosen a file). E.g.::

      sqlUpdate={{UPDATE Note SET pathFileName = '{{fileDestination}}' WHERE id={{slaveId}} LIMIT 1}}

  * *sqlDelete* = `{{<query>}}` - fired if `slaveId>0` and no upload exist (user has not choosen a file). E.g.::

      sqlDelete={{DELETE FROM Note WHERE id={{slaveId:V}}  LIMIT 1}}

  * *sqlAfter* = `{{<query>}}` - fired after all previous queries have been fired. Might update the new created id to a primary record. E.g.::

      sqlAfter={{UPDATE Person SET noteIdPicture = {{slaveId}} WHERE id={{id:R0}} LIMIT 1 }}


.. _split-pdf-upload:

Split PDF Upload
;;;;;;;;;;;;;;;;

Additional to the upload, it's possible to split the uploaded file (only PDF files) into several SVG files, one file per
page. The split is done via http://www.cityinthesky.co.uk/opensource/pdf2svg/.

 * *FormElement.parameter*:

   * *fileSplit* = `<type>` - Activate the splitting process. Only possible value: `fileSplit=svg`.
   * *fileDestinationSplit* = `<pathFileName (pattern)>` - Target directory and filename pattern for the created & split'ed files. E.g. ::

       fileDestinationSplit = fileadmin/protected/{{id:R}}.{{filenameBase}}.%02d.svg

   * *tableNameSplit* = `<tablename>` - Reference in table 'Split' to the table, which holds the original PDF file.

The splitting happens immediately after the user pressed save.

To easily access the split files via QFQ, per file per record is created in table 'Split'.

Table 'Split':

+--------------+--------------------------------------------------------------------------------------------+
| Column       | Description                                                                                |
+==============+============================================================================================+
| id           | Uniq auto increment index                                                                  |
+--------------+--------------------------------------------------------------------------------------------+
| tableName    | Name of the table, where the reference to the original file (multipage PDF file) is saved. |
+--------------+--------------------------------------------------------------------------------------------+
| xId          | Primary id of the reference record.                                                        |
+--------------+--------------------------------------------------------------------------------------------+
| pathFileName | Path/filename reference to one of the created files                                        |
+--------------+--------------------------------------------------------------------------------------------+
| created      | Timestamp                                                                                  |
+--------------+--------------------------------------------------------------------------------------------+

One usecase why to split an upload: annotate individual pages by using the `FormElement`.type=`annotate`.

.. _class-action:

Class: Action
-------------

Type: before... | after...
^^^^^^^^^^^^^^^^^^^^^^^^^^

These type of 'action' *FormElements* will be used to implement data validation or creating/updating additional records.

Types:

  * beforeLoad (e.g. good to check access permission)
  * afterLoad
  * beforeSave (e.g. to prohibit creating of duplicate records)
  * afterSave (e.g. to to create & update additional records)
  * beforeInsert
  * afterInsert
  * beforeUpdate
  * afterUpdate
  * beforeDelete (e.g. to delete slave records)
  * afterDelete
  * paste (configure copy/paste forms)

Parameter: sqlValidate
''''''''''''''''''''''

  Perform checks by fireing a SQL query and expecting a predefined number of selected records.

  * OK: the `expectRecords` number of records has been selected. Continue processing the next *FormElement*.
  * Fail: the `expectRecords` number of records has not been selected (less or more): Display the error message
    `messageFail` and abort the whole (!) current form load or save.

  *FormElement.parameter*:

  * *requiredList* = `<fe.name[s]>` - List of `native`-*FormElement* names: only if all of those elements are filled (!=0 and !=''), the *current*
    `action`-*FormElement* will be processed. This will enable or disable the check, based on the user input! If no
    `native`-*FormElement* names are given, the specified check will always be performed.
  * *sqlValidate* = `{{<query>}}` - validation query. E.g.: `sqlValidate={{!SELECT id FROM Person AS p WHERE p.name LIKE {{name:F:all}} AND p.firstname LIKE {{firstname:F:all}} }}`

    * Pay attention to `{{!...` after the equal sign.

  * *expectRecords* = `<value>`- number of expected records.

    * *expectRecords* = `0` or *expectRecords* = `0,1` or *expectRecords* = `{{SELECT COUNT(id) FROM Person}}`
    * Separate multiple valid record numbers by ','. If at least one of those matches, the check will pass successfully.

  * *messageFail* = `<string>` - Message to show. E.g.: *messageFail* = `There is already a person called {{firstname:F:all}} {{name:F:all}}`

.. _slave-id:

Parameter: slaveId
''''''''''''''''''

*FormElement.parameter*:

  * *slaveId* = `<id>`:

    * Auto fill: name the action `action`-*FormElement* equal to an existing column (table from the current form definition).
      *slaveId* will be automatically filled with the value of the named column.

      * If there is no such named column name, set *slaveId* = `0`.

    * Explicit definition: *slaveId* = `123` or *slaveId* = `{{SELECT id ...}}`

Note:

  * `{{slaveId}}` can be used in any query of the current *FormElement*.
  * If the `action`-*FormElement* name exist as a column in the master record: Update that column *automatically* with the
    recent slaveId
  * After an INSERT the `last_insert_id()` becomes the *slaveId*).

Parameter: sqlBefore / sqlInsert / sqlUpdate / sqlDelete / sqlAfter
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

  * Save values of a form to different record(s), optionally on different table(s).
  * Typically useful on 'afterSave' - be careful when using it earlier, e.g. beforeLoad.

*FormElement.parameter*:

  * *requiredList* = `<fe.name[s]>` - List of `native`-*FormElement*: only if all of those elements are filled, the current
    `action`-*FormElement* will be processed.

  * *sqlBefore* = `{{<query>}}` - always fired (before any *sqlInsert*, *sqlUpdate*, ..)
  * *sqlInsert* = `{{<query>}}` - fired if *slaveId* == `0` or *slaveId* == `''`.
  * *sqlUpdate* = `{{<query>}}` - fired if *slaveId* > `0`.
  * *sqlDelete* = `{{<query>}}` - fired if *slaveId* > `0`, after *sqlInsert* or *sqlUpdate*. Be careful not to delete filled records!
    Always add a check, if values given, not to delete the record! *sqlHonorFormElements* helps to skip such checks.
  * *sqlAfter* = `{{<query>}}` - always fired (after *sqlInsert*, *sqlUpdate* or *sqlDelete*).
  * *sqlHonorFormElements* = `<fe.name[s]>` list of *FormElement* names (this parameter is optional).

    * If one of the named *FormElements* is not empty:

      * fire *sqlInsert* if *slaveId* == `0`,
      * fire *sqlUpdate* if *slaveId* > `0`

    * If all of the named *FormElements* are empty:

      * fire *sqlDelete* if *slaveId* > `0`


Example
'''''''

Situation 1: master.xId=slave.id (1:1)

 * Name the action element 'xId': than {{slaveId}} will be automatically set to the value of 'master.xId'

   * {{slaveId}} == 0 ? 'sqlInsert' will be fired.
   * {{slaveId}} != 0 ? 'sqlUpdate' will be fired.

 * In case of firing 'sqlInsert', the 'slave.id' of the new created record are copied to master.xId (the database will
   be updated automatically).

 * If the automatic update of the master record is not suitable, the action element should have no name or a name
   which does not exist as a column of the master record. Define  `slaveId={{SELECT id ...}}`

 * Two *FormElements*  `myStreet` and `myCity`:

    * Without *sqlHonorFormElements*. Parameter: ::

       sqlInsert = {{INSERT INTO address (`street`, `city`) VALUES ('{{myStreet:FE:alnumx:s}}', '{{myCity:FE:alnumx:s}}') }}
       sqlUpdate = {{UPDATE address SET `street` = '{{myStreet:FE:alnumx:s}}', `city` = '{{myCity:FE:alnumx:s}}'  WHERE id={{slaveId}} LIMIT 1 }}
       sqlDelete = {{DELETE FROM address WHERE id={{slaveId}} AND '{{myStreet:FE:alnumx:s}}'='' AND '{{myCity:FE:alnumx:s}}'='' LIMIT 1 }}

    * With *sqlHonorFormElements*. Parameter: ::

       sqlHonorFormElements = myStreet, myCity     # Non Templategroup
       sqlInsert = {{INSERT INTO address (`street`, `city`) VALUES ('{{myStreet:FE:alnumx:s}}', '{{myCity:FE:alnumx:s}}') }}
       sqlUpdate = {{UPDATE address SET `street` = '{{myStreet:FE:alnumx:s}}', `city` = '{{myCity:FE:alnumx:s}}'  WHERE id={{slaveId}} LIMIT 1 }}
       sqlDelete = {{DELETE FROM address WHERE id={{slaveId}} LIMIT 1 }}

       # For Templategroups: sqlHonorFormElements = myStreet%d, myCity%d

Situation 2: master.id=slave.xId (1:n)

 * Name the action element *different* to any column name of the master record (or no name).
 * Determine the slaveId: `slaveId={{SELECT id FROM slave WHERE slave.xxx={{...}} LIMIT 1}}`

   * {{slaveId}} == 0 ? 'sqlInsert' will be fired.
   * {{slaveId}} != 0 ? 'sqlUpdate' will be fired.

 * Two *FormElements*  `myStreet` and `myCity`. The `person` is the master record, `address` is the slave:

    * Without *sqlHonorFormElements*. Parameter: ::

       slaveId = {{SELECT id FROM address WHERE personId={{id}} ORDER BY id LIMIT 1 }}
       sqlInsert = {{INSERT INTO address (`personId`, `street`, `city`) VALUES ({{id}}, '{{myStreet:FE:alnumx:s}}', '{{myCity:FE:alnumx:s}}') }}
       sqlUpdate = {{UPDATE address SET `street` = '{{myStreet:FE:alnumx:s}}', `city` = '{{myCity:FE:alnumx:s}}'  WHERE id={{slaveId}} LIMIT 1 }}
       sqlDelete = {{DELETE FROM address WHERE id={{slaveId}} AND '{{myStreet:FE:alnumx:s}}'='' AND '{{myCity:FE:alnumx:s}}'='' LIMIT 1 }}

    * With *sqlHonorFormElements*. Parameter: ::

       slaveId = {{SELECT id FROM address WHERE personId={{id}} ORDER BY id LIMIT 1 }}
       sqlHonorFormElements = myStreet, myCity       # Non Templategroup
       sqlInsert = {{INSERT INTO address (`personId`, `street`, `city`) VALUES ({{id}}, '{{myStreet:FE:alnumx:s}}', '{{myCity:FE:alnumx:s}}') }}
       sqlUpdate = {{UPDATE address SET `street` = '{{myStreet:FE:alnumx:s}}', `city` = '{{myCity:FE:alnumx:s}}'  WHERE id={{slaveId}} LIMIT 1 }}
       sqlDelete = {{DELETE FROM address WHERE id={{slaveId}} LIMIT 1 }}

       # For Templategroups: sqlHonorFormElements = myStreet%d, myCity%d

Type: sendmail
^^^^^^^^^^^^^^

* Send mail(s) will be processed after:

   * saving the record ,
   * processing all uploads,
   * processing `afterSave` action `FormElements`.


* *FormElement.value* = `<string>` - Body of the email. See also: `html-formatting`_

* *FormElement.parameter*:

  * *sendMailTo* = `<string>` - Comma-separated list of receiver email addresses. Optional: 'realname <john@doe.com>. If there
    is no recipient email address, *no* mail will be sent.
  * *sendMailCc* = `<string>` - Comma-separated list of receiver email addresses. Optional: 'realname <john@doe.com>.
  * *sendMailBcc* = `<string>` - Comma-separated list of receiver email addresses. Optional: 'realname <john@doe.com>.
  * *sendMailFrom* = `<string>` - Sender of the email. Optional: 'realname <john@doe.com>'. **Mandatory**.
  * *sendMailSubject* = `<string>` - Subject of the email.
  * *sendMailReplyTo* = `<string>` - Reply this email address. Optional: 'realname <john@doe.com>'.
  * *sendMailAttachment* = `<string>` - List of 'sources' to attach to the mail as files. Check `attachment`_ for options.
  * *sendMailHeader* = `<string>` - Specify custom header.
  * *sendMailFlagAutoSubmit* = `<string>` - **on|off** - If 'on' (default), the mail contains the header
    'Auto-Submitted: auto-send' - this suppress a) OoO replies, b) forwarding of emails.
  * *sendMailGrId* = `<string>` - Will be copied to the mailLog record. Helps to setup specific logfile queries.
  * *sendMailXId* = `<string>` - Will be copied to the mailLog record. Helps to setup specific logfile queries.
  * *sendMailXId2* = `<string>` - Will be copied to the mailLog record. Helps to setup specific logfile queries.
  * *sendMailXId3* = `<string>` - Will be copied to the mailLog record. Helps to setup specific logfile queries.
  * *sendMailMode* = `<string>` - **html** - if set, the e-mail body will be rendered as html.
  * *sendMailSubjectHtmlEntity* = `<string>` - **encode|decode|none** - the mail subject will be htmlspecialchar() encoded / decoded (default) or none (untouched).
  * *sendMailBodyHtmlEntity*= `<string>`  - **encode|decode|none** - the mail body will be htmlspecialchar() encoded, decoded (default) or none (untouched).

* To use values of the submitted form, use the STORE_FORM. E.g. `{{name:F:allbut}}`
* To use the `id` of a new created or already existing primary record, use the STORE_RECORD. E.g. `{{id:R}}`.
* By default, QFQ stores values 'htmlspecialchars()' encoded. If such values have to send by email, the html entities are
  unwanted. Therefore the default setting for 'subject' und 'body' is to decode the values via 'htmlspecialchars_decode()'.
  If this is not wished, it can be turned off by `sendMailSubjectHtmlEntity=none` and/or `sendMailBodyHtmlEntity=none`.

* For debugging, please check `REDIRECT_ALL_MAIL_TO`_.

Example to attach one file1.pdf (with the attachment filename 'readme.pdf') and concatenate two PDF, created on the fly
from the www.example.com and ?export (with the attachment filename 'personal.pdf'): ::

  sendMailAttachmemt = F:fileadmin/file1.pdf|d:readme.pdf|C|u:http://www.example.com|p:?id=export&r=123&_sip=1|d:personal.pdf

Type: paste
'''''''''''

See also `copy-form`_.

* *sql1* = `{{<query>}}` - e.g. `{{!SELECT {{id:P}} AS id, '{{myNewName:FE:allbut}}' AS name}}` (only one record) or `{{!SELECT i.id AS id, {{basketId:P}} AS basketId FROM Item AS i WHERE i.basketId={{id:P}} }}` (multiple records)

  * Pay attention to '!'.
  * For every row, a new record is created in `recordDestinationTable`.
  * Column 'id' is not copied.
  * The `recordSourceTable` together with column `id` will identify the source record.
  * Columns not specified, will be copied 1:1 from source to destination.
  * Columns specified, will overwrite the source value.

* *FormElement.parameter*:

  * *recordSourceTable* = `<tableName>` - Optional: table from where the records will be copied. Default: <recordDestinationTable>
  * *recordDestinationTable* = `<tableName>`  - table where the new records will be copied to.
  * *translateIdColumn* = `<column name>`  - column name to update references of newly created id's.

.. _form-magic:

Form Magic
----------

Parameter
^^^^^^^^^

* Table column `id`: QFQ expect that each table, which will be loaded in a form, contains an autoincrement column of name `id`.
  It's not necessary to create a FormElement `id` in a form - but it won't disturb.

* Parameter (one or more) in the SIP url, which exist as a column in the form table (SIP parameter name is equal to a table column name),
  will be automatically saved in the record. This acts as 'hidden magic'.

  Example: A slave record (e.g. an address of a person) has to be assigned to a master record (a person). Just give the
  `pId` in the link who calls the address form. The following creates a 'new' button for an address for all persons, and
  the pId will be automatically saved in the address table: ::

		SELECT CONCAT('{{pageAlias:T}}&form=address&r=0&pId=', p.id) AS _pagen FROM Person AS p

  Such parameter, which the form expects to be in the SIP url, should be specified in Form.permitNew and/or Form.permitEdit.
  It's only a check for the webmaster, not to forgot a parameter in a SIP url.

* FormElement.type = subrecord

  Subrecord's will automatically create `new`, `edit` and `delete` links. To inject parameter in those automatically created
  links, use `FormElement.parameter.detail` . See subrecord-option_.


* FormElement.type = extra

  If a table column should be saved with a specific value, and the value should not be shown to the user, the FE.type='extra'
  will do the job. The value could be static or calculated on the fly. Often it's easier to specify such a parameter/value
  in the SIP url, but if the form is called from multiple places, an `extra` element is more suitable.

Variables
^^^^^^^^^

* Form.parameter.fillStoreVar / FormElement.parameter.fillStoreVar

  A SQL statement will fill STORE_VARS. Such values can be used during form load and/or save.

Action
^^^^^^

* Action FE

  Via `FormElement.parameter.requiredList` an element can be enabled / disabled, depending of a user provided input
  in one of the specified required FEs.


.. _multi-language-form:

Multi Language Form
-------------------

QFQ Forms might be configured for up to 5 different languages. Per language there is one extra field in the *Form editor*.
Which field represents which language is configured in configuration_.

* The Typo3 installation needs to be configured to handle different languages - this is independent of QFQ and not covered
  here. QFQ will use the Typo3 internal variable 'pageLanguage', which typically correlates to the URL parameter 'L' in the URL.
* In configuration_ the Typo3 language index (value of 'L') and a language label have to be configured for each language.
  Only than, the additional language fields in the *Form editor* will be shown.

Example
^^^^^^^

Assuming the Typo3 page has the

* default language, L=0
* English, L=1
* Spanish, L=2

Configuration in configuration_: ::

		formLanguageAId = 1
		formLanguageALabel = English

		formLanguageBId = 2
		formLanguageBLabel = Spanish

The default language is not covered in configuration_.

The *Form editor* now shows on the pill 'Basic' (Form and FormEditor) for both languages each an additional parameter
input field. Any input field in the *Form editor* can be redeclared in the corresponding language parameter field. Any
missing definition means 'take the default'. E.g.:

* Form: 'person'

	+--------------------+--------------------------+
	| Column             | Value                    |
	+====================+==========================+
	| title              | Eingabe Person           |
	+--------------------+--------------------------+
	| languageParameterA | title=Input Person       |
	+--------------------+--------------------------+
	| languageParameterB | title=Persona de entrada |
	+--------------------+--------------------------+

* FormElement 'firstname' in Form 'person':

	+--------------------+------------------------------------------------+
	| Column             | Value                                          |
	+====================+================================================+
	| title              | Vorname                                        |
	+--------------------+------------------------------------------------+
	| note               | Bitte alle Vornamen erfassen                   |
	+--------------------+------------------------------------------------+
	| languageParameterA | | title=Firstname                              |
	|                    | | note=Please give all firstnames              |
	+--------------------+------------------------------------------------+
	| languageParameterB | | title=Persona de entrada                     |
	|                    | | note=Por favor, introduzca todos los nombres |
	+--------------------+------------------------------------------------+


The following fields are possible:

* Form: *title, showButton, forwardMode, forwardPage, bsLabelColumns, bsInputColumns, bsNoteColumns, recordLockTimeoutSeconds*
* FormElement: *label, mode, modeSql, class, type, subrecordOption, encode, checkType, ord, tabindex, size, maxLength,*
  *bsLabelColumns, bsInputColumns, bsNoteColumns,rowLabelInputNote, note, tooltip, placeholder, value, sql1, feGroup*

.. _dynamic-update:

Dynamic Update
--------------

The 'Dynamic Update' feature makes a form more interactive. If a user changes a *FormElement* who is tagged with
'dynamicUpdate', *all* elements who are tagged with 'dynamicUpdate', will be recalculated and rerendered.

The following fields will be recalculated during 'Dynamic Update'

* 'modeSql' - Possible values: 'show', 'required', 'readonly', 'hidden'
* 'label'
* 'value'
* 'note'
* 'parameter.*' - especially 'itemList'

To make a form dynamic:

* Mark all *FormElements* with `dynamic update`=`enabled`, which should **initiate** or **receive** updates.

See #3426 / Dynamic Update: Inputs loose the new content and shows the old value:

* On **all** `dynamic update` *FormElements* an explicit definition of `value`, including a sanitize class, is necessary
  (except the field is numeric). **A missing definition let's the content overwrite all the time with the old value**.
  A typical definition for `value` looks like (default store priority is: FSRVD)::

     {{<FormElement name>::alnumx}}

* Define the receiving *FormElements* in a way, that they will interpret the recent user change! The form variable of the
  specific sender *FormElement* `{{<sender element>:F:<sanitize>}}` should be part of one of the above fields to get an
  impact. E.g.:
  ::

    [receiving *FormElement*].parameter: itemList={{ SELECT IF({{carPriceRange:FE:alnumx}}='expensive','Ferrari,Tesla,Jaguar','General Motors,Honda,Seat,Fiat') }}

  Remember to specify a 'sanitize' class - a missing sanitize class means 'digit', every content, which is not numeric,
  violates the sanitize class and becomes therefore an empty string!

* If the dynamic update should work on existing and *new* records, it's important to guarantee that the query result is not empty!
  even if the primary record does not exist! E.g. use a `LEFT JOIN`. The following query is ok for `new` and `edit`. ::

    {{SELECT IF( IFNULL(adr.type,'') LIKE '%token%','show','hidden') FROM (SELECT 1) AS fake LEFT JOIN Address AS adr ON adr.type='{{type:FR0}}' LIMIT 1}}

Examples
^^^^^^^^

* Master FormElement 'music' is a radio/enum of 'classic', 'jazz', 'pop'.

Content of a select list
''''''''''''''''''''''''

* Slave FormElement 'interpret' is 'select'-list, depending of 'music'

::

   sql={{!SELECT name FROM interpret WHERE music={{music:FE:alnumx}} ORDER BY name}}

Show / Hide a *FormElement*
'''''''''''''''''''''''''''

* Slave 'interpret' is displayed only for 'pop'. Field 'modeSql':

::

    {{SELECT IF( '{{music:FR:alnumx}}'='pop' ,'show', 'hidden' ) }}

.. _form-layout:

Form Layout
-----------

The forms will be rendered with Bootstrap CSS classes, based on the 12 column grid model (Bootstrap 3.x).
Generally a 3 column layout for *label* columns on the left side, an *input* field column in the middle and a *note*
column on the right side will be rendered.

The used default column (=bootstrap grid) width is *3,6,3* for *label, input, note*.

* The system wide default can be changed via `configuration`_ - the new settings are the default
  settings for all forms.
* Per *Form* settings can be done in the *Form* parameter field. They overwrite the system wide default.
* Per *FormElement* settings can be done in the *FormElement* parameter field. They overwrite the *Form* setting.

A column will be switched off (no wrapping via `<div class='col-md-?>`) by setting a `0` on the respective column.

Custom field width
^^^^^^^^^^^^^^^^^^

Per *FormElement* set `BS Label Columns`, `BS Input Columns` or `BS Note Columns` to customize an individual width.
The sum of these three columns should always be 12.

Multiple Elements per row
^^^^^^^^^^^^^^^^^^^^^^^^^

Every row is by default wrapped in a `<div class='form-group'>` and every column is wrapped in a `<div class='col-md-?'>`.
To display multiple input elements in one row, the wrapping of the *FormElement* row and of the three columns can be
customized via the checkboxes of `Label / Input / Note`. Every open and every close tag can be individually switched on
or off.

E.g. to display 2 *FormElements* in a row with one label (first *FormElement*) and one note (last *FormElement*) we need
the following (switch off all non named):

* First *FormElement*

  * open row tag: `row` ,
  * open and close label tag: `label`, `/label`,
  * open and close field tag: `input`, `/input`,

* Second *FormElement*

  * open and close field tag: `input`, `/input`,
  * open and close note tag: `note`, `/note`,
  * close row tag: `/row` ,

.. _`copy-form`:

Copy Form
---------

Records (=master) and child records can be duplicated (=copied) by a regular `Form`, extended by `FormElements` of type 'paste'.
A 'copy form' works either in:

* 'copy and paste now' mode: the 'select' and 'paste' `Form` is merged in one form, only one master record is possible,
* 'copy now, paste later' mode: the 'select' `Form` selects master record(s), the 'paste' Form paste's them later.

Concept
^^^^^^^

A 'select action' (e.g. a `Form` or a button click) creates record(s) in the table `Clipboard`. Each clipboard record contains:

* the 'id(s)' of the record(s) to duplicate,
* the 'paste' form id (that `Form` defines, to which table the master records belongs to, as well as rules of how to
  duplicate any slave records) and where to copy the new records
* user identifier (QFQ cookie) to separate clipboard records of different users inside the Clipboard table.

The 'select action' is also responsible to delete old clipboard records of the current user, before new clipboard records are
created.

The 'paste form' iterates over all master record id(s) in the `Clipboard` table. For each master record id, all FormElements
of type `paste` are fired (incl. the creating of slave records).

E.g. if there is a basket with different items and you want to duplicate the whole basket including new items, create a
form with the following parameter

 * Form

   * Name: `copyBasket`
   * Table: `Clipboard`
   * Show Button: only `close` and `save`

 * FormElement 1: Record id of the source record.

   * Name: `idSrc`
   * Lable: `Source Form`
   * Class: `native`
   * Type: `select`
   * sql1: `{{! SELECT id, title FROM Basket }}`

 * FormElement 2: New name of the copied record.

   * Name: `myNewName`
   * Class: `native`
   * Type: `text`

 * FormElement 3: a) Check that there is no name conflict. b)Purge any old clipboard content of the current user.

   * Name: `clearClipboard`
   * Class: `action`
   * Type: `beforeSave`
   * Parameter:

     * `sqlValidate={{!SELECT f.id FROM Form AS f WHERE f.name LIKE '{{myName:FE:alnumx}}' LIMIT 1}}`
     * `expectRecords = 0`
     * `messageFail = There is already a form with this name`
     * `sqlAfter={{DELETE FROM Clipboard WHERE cookie='{{cookieQfq:C0:alnumx}}' }}`

 * FormElement 4: Update the clipboard source reference, with current {{cookieQfq:C}} identifier.

   * Name: `updateClipboardRecord`
   * Class: `action`
   * Type: `afterSave`
   * Parameter: `sqlAfter={{UPDATE Clipboard SET cookie='{{cookieQfq:C0:alnumx}}', formIdPaste={{formId:S0}} /* PasteForm */  WHERE id={{id:R}} LIMIT 1 }}`

 * FormElement 5: Copy basket identifier.

   * Name: `basketId`
   * Class: `action`
   * Type: `paste`
   * sql1: `{{!SELECT {{id:P}} AS id,  '{{myNewName:FE:allbut}}' AS name}}`
   * Parameter: `recordDestinationTable=Basket`

 * FormElement 6: Copy items of basket.

   * Name: `itemId`
   * Class: `action`
   * Type: `paste`
   * sql1: `{{!SELECT i.id AS id, {{basketId:P}} AS basketId FROM Item AS i WHERE i.basketId={{id:P}} }}`
   * Parameter: `recordDestinationTable=Item`


Table self referencing records
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Records might contain references to other records in the same table. E.g. native FormElements might assigned to a fieldSet,
templateGroup or pill, a fieldSet might assigned to other fieldsets or pills and so on. When duplicating a `Form` and the
corresponding `FormElements` all internal references needs to be updated as well.

On each FormElement.type=`paste` record, the column to be updated is defined via:

 * parameter: translateIdColumn = <column name>

For the 'copyForm' this would be 'feIdContainer'.

The update of the records is started after all records have been copied (of the specific FormElement.type=`paste` record).

.. _delete-record:

Delete Record
-------------

Deleting record(s) via QFQ might be solved by either:

* using the `delete` button on a form on the top right corner.
* by letting `report`_ creating a special link (see below). The link contains the record id and:

  * a form name, or
  * a table name.

Deleting a record just by specifying a table name, will only delete the defined record (no slave records).

* By using a delete button via `report` or in a `subrecord` row, a ajax request is send.
* By using a delete button on the top right corner of the form, the form will be closed after deleting the record.

Example for report::

   SELECT p.name, CONCAT('form=person&r=', p.id) AS _Paged FROM Person AS p
   SELECT p.name, CONCAT('table=Person&r=', p.id) AS _Paged FROM Person AS p

To automatically delete slave records, use a form and create `beforeDelete` FormElement(s) on the form:

  * class: action
  * type: beforeDelete
  * parameter: sqlAfter={{DELETE FROM <slaveTable> WHERE <slaveTable>.<masteId>={{id:R}} }}

You might also check the form 'form' how the slave records 'FormElement' will be deleted.

.. _locking-record:

Locking Record / Form
---------------------

Support for record locking is given with mode:

* *exclusive*: user can't force a write.

  * Including a timeout (default 15 mins dirtyRecordTimeoutSeconds in configuration_) for maximum lock time.

* *advisory*: user is only warned, but allowed to overwrite.
* *none*: no bookkeeping about locks.

For 'new' records (r=0) there is no locking at all.

The record locking protection is based on the `tablename` and the `record id`. Different `Forms`, with the same primary table,
will be protected by record locking. On the other side, action-`FormElements` updating non primary table records are not
protected by 'record locking': the QFQ record locking is *NOT 100%*.

The 'record locking' mode will be specified per `Form`. If there are multiple Forms with different modes, and there is
already a lock for a `tablename` / `record id` pair, the most restrictive will be applied.


Best practice
-------------

Custom default value only for 'new records'
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Method 1
''''''''

On `Form.parameter` define a `fillStoreVar` query with a column name equal to a form field. That's all.

Example: ::

  FormElement.name = technicalContact
  Form.parameter.fillStoreVar = {{!SELECT CONCAT(p.firstName, ' ', p.name) AS technicalContact FROM Person AS p WHERE p.account='{{feUser:T}}' }}

What we use here is the default STORE prio FSRVD. If the form loads with r=0, 'F', 'S' and 'R' are empty. 'V' is filled.
If r>0, than 'F' and 'S' are empty and 'R' is filled.

Method 2
''''''''
In the specific `FormElement` set `value={{columnName:RSE}}`. The link to the form should be rendered with
'"...&columnName=<data>&..." AS _page'. The trick is that the STORE_RECORD is empty for new records, and therefore the
corresponding value from STORE_SIP will be returned. Existing records will use the already saved value.

Central configured values
^^^^^^^^^^^^^^^^^^^^^^^^^

Any variable in configuration_ can be used by *{{<varname>:Y}}* in form or report statements.

E.g.

  TECHNICAL_CONTACT = jane.doe@example.net

Could be used in an *FormElement.type* = sendmail with *parameter*  setting *sendMailFrom={{TECHNICAL_CONTACT:Y}}*.

Debug Report
^^^^^^^^^^^^

Writing "report's" in the nested notation or long queries broken over several lines, might not interpreted as wished.
Best for debugging is to specify in the tt-content record::

  debugShowBodyText = 1

Note: Debug information is only display if it's enabled in  configuration_ by

 * *showDebugInfo: yes* or
 * *showDebugInfo: auto* and logged in in the same Browser as a Typo3 backend user.

More detailed error messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If *showDebugInfo* is enabled, a full stacktrace and variable contents are displayed in case of an error.

Form search
^^^^^^^^^^^

QFQ content record::

  # Creates a small form that redirects back to this page
  10 {
    sql = SELECT '_'
    head = <form action='#' method='get'><input type='hidden' name='id' value='{{pageId:T}}'>Search: <input type='text' name='search' value='{{search:CE:all}}'><input type='submit' value='Submit'></form>
  }

  # SQL statement will find and list all the relevant forms - be careful not to open a cross site scripting door: the parameter 'search' needs to be sanitized.
  20 {
    sql = SELECT CONCAT('?detail&form=form&r=', f.id) AS _Pagee, f.id, f.name, f.title
              FROM Form AS f
              WHERE f.name LIKE  '%{{search:CE:alnumx}}%'
    head = <table class='table'>
    tail = </table>
    rbeg = <tr>
    rend = </tr>
    fbeg = <td>
    fend = </td>
  }


Form: compute next free 'ord' automatically
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Requirement: new records should automatically get the highest number plus 10 for their 'ord' value. Existing records
should not be altered.

Version 1
'''''''''

Compute the next 'ord' in advance in the subrecord field of the primary form. Submit that value to the new record
via SIP parameter to the secondary form.

On the secondary form: for 'new' records choose the computed value, for existing records leave the value
unchanged.

* Master form, `subrecord` *FormElement*, field `parameter`: set ::

    detail=id:formId,{{SELECT '&', IFNULL(fe.ord,0)+10 FROM Form AS f LEFT JOIN *FormElement* AS fe ON fe.formId=f.id WHERE
    f.id={{r:S0}} ORDER BY fe.ord DESC LIMIT 1}}:ord


* Slave form, `ord` *FormElement*, field `value`: set

  ::

   `{{ord:RS0}}`.

Version 2
'''''''''

Compute the next 'ord' as default value direct inside the secondary form. No change is needed for the primary form.

* Secondary form, `ord` *FormElement*, field `value`: set `{{SELECT IF({{ord:R0}}=0,  MAX(IFNULL(fe.ord,0))+10,{{ord:R0}})  FROM (SELECT 1) AS a LEFT JOIN FormElement AS fe ON fe.formId={{formId:S0}} GROUP BY fe.formId}}`.

Form: Person Wizard - firstname, city
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Requirement: A form that displays the column 'firstname' from table 'Person' and 'city' from table 'Address'. If the
records not exist, the form should create it.

Form primary table: Person

Form slave table: Address

Relation: `Person.id = Address.personId`

* Form: wizard

  * Name: wizard
  * Title: Person Wizard
  * Table: Person
  * Render: bootstrap

* *FormElement*: firstname

  * Class: **native**
  * Type: **text**
  * Name: firstname
  * Label: Firstname

* *FormElement*: email, text, 20

  * Class: **native**
  * Type: **text**
  * Name: city
  * Label: City
  * Value: `{{SELECT city FROM Address WHERE personId={{r}} ORDER BY id LIMIT 1}}`

* *FormElement*: insert/update address record

  * Class: **action**
  * Type: **afterSave**
  * Label: Manage Address
  * Parameter:

    * `slaveId={{SELECT id FROM Address WHERE personId={{r}} ORDER BY id LIMIT 1}}`
    * `sqlInsert={{INSERT INTO Address (personId, city) VALUES ({{r}}, '{{city:F:allbut:s}}') }}`
    * `sqlUpdate={{UPDATE Address SET city='{{city:F:allbut:s}}' WHERE id={{slaveId:V}} }}`
    * `sqlDelete={{DELETE FROM Address WHERE id={{slaveId:V}} AND ''='{{city:F:allbut:s}}' LIMIT 1}}`

Form: Person Wizard - firstname, single note
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Requirement: A form that displays the column 'firstname' from table 'Person' and 'note' from table 'Note'.
If the records don't exist, the form should create it.
Column Person.noteId points to Note.id

Form primary table: Person

Form slave table: Address

Relation: `Person.id = Address.personId`

* Form: wizard

  * Name: wizard
  * Title: Person Wizard
  * Table: Person
  * Render: bootstrap

* *FormElement*: firstname

  * Class: **native**
  * Type: **text**
  * Name: firstname
  * Label: Firstname

* *FormElement*: email, text, 20

  * Class: **native**
  * Type: **text**
  * Name: note
  * Label: Note
  * Value: `{{SELECT Note FROM Note AS n, Person AS p WHERE p.id={{r}} AND p.noteId=n.id ORDER BY id }}`

* *FormElement*: insert/update address record

  * Class: **action**
  * Type: **afterSave**
  * Name: noteId
  * Label: Manage Note
  * Parameter:

    * `sqlInsert={{INSERT INTO Note (note) VALUES ('{{note:F:allbut:s}}') }}`
    * `sqlUpdate={{UPDATE Note SET note='{{note:F:allbut:s}}' WHERE id={{slaveId:V}} }}`

.. _example_class_template_group:

Icons Template Group
^^^^^^^^^^^^^^^^^^^^

This example will display graphics instead of text 'add' and 'remove'. Also there is a distance between the templateGroups.

 * FormElement.parameter::

     tgClass = qfq-child-margin-top
     tgAddClass = btn alert-success
     tgAddText = <span class="glyphicon glyphicon-plus" aria-hidden="true"></span>
     tgRemoveClass = btn btn-danger alert-danger
     tgRemoveText = <span class="glyphicon glyphicon-remove" aria-hidden="true"></span>

Chart
^^^^^

* QFQ delivers a chart JavaScript lib: https://github.com/nnnick/Chart.js.git. Docs: http://www.chartjs.org/docs/
* The library is not sourced in the HTML page automatically. To do it, either include the lib
  `typo3conf/ext/qfq/Resources/Public/JavaScript/Chart.min.js`:

  * in the specific tt_content record (shown below in the example) or
  * system wide via Typo3 Template record.

* By splitting HTML and JavaScript code over several lines, take care not accidentally to create a 'nesting'-end token.
  Check the line after `10.tail =`. It's '}' alone on one line. This is a valid 'nesting'-end token!. There are two options
  to circumvent this:

  * Don't nest the HTML & JavaScript code - bad workaround, this is not human readable.
  * Select different nesting token, e.g. '<' (check the first line on the following example).

::

     # <

     10.sql = SELECT '_'
     10.head =
       <div style="height: 1024px; width: 640px;">
         <h3>Distribution of FormElement types over all forms</h3>
         <canvas id="barchart" width="1240" height="640"></canvas>
       </div>
       <script src="typo3conf/ext/qfq/Resources/Public/JavaScript/Chart.min.js"></script>
       <script>
         $(function () {
           var ctx = document.getElementById("barchart");
           var barChart = new Chart(ctx, {
             type: 'bar',
               data: {

     10.tail =
               }
           });
         });
       </script>

     # Labels
     10.10 <
       sql = SELECT "'", fe.type, "'" FROM FormElement AS fe GROUP BY fe.type ORDER BY fe.type
       head = labels: [
       tail = ],
       rsep = ,
     >

     # Data
     10.20 <
       sql = SELECT COUNT(fe.id) FROM FormElement AS fe GROUP BY fe.type ORDER BY fe.type
       head = datasets: [ {   data: [
       tail = ],  backgroundColor: "steelblue", label: "FormElements" } ]
       rsep = ,
     >

Upload Form Simple
^^^^^^^^^^^^^^^^^^

Table Person

+---------------------+--------------+
| Name                | Type         |
+=====================+==============+
| id                  | int          |
+---------------------+--------------+
| name                | varchar(255) |
+---------------------+--------------+
| pathFileNamePicture | varchar(255) |
+---------------------+--------------+
| pathFileNameAvatar  | varchar(255) |
+---------------------+--------------+

* Form:

  * Name: UploadSimple
  * Table: Person

* FormElements:

  * Name: name

    * Type: text
    * Label: Name

  * Name: pathFileNamePicture

    * Type: upload
    * Label: Picture
    * Parameter::

        fileDestination=fileadmin/user/{{id:R0}}-picture-{{filename}}

  * Name: pathFileNameAvatar

    * Type: upload
    * Label: Avatar
    * Parameter::

        fileDestination=fileadmin/user/{{id:R0}}-avatar-{{filename}}


Upload Form Advanced 1
^^^^^^^^^^^^^^^^^^^^^^

Table: Person

 +---------------------+--------------+
 | Name                | Type         |
 +=====================+==============+
 | id                  | int          |
 +---------------------+--------------+
 | name                | varchar(255) |
 +---------------------+--------------+

Table: Note

 +---------------------+--------------+
 | Name                | Type         |
 +=====================+==============+
 | id                  | int          |
 +---------------------+--------------+
 | pId                 | int          |
 +---------------------+--------------+
 | type                | varchar(255) |
 +---------------------+--------------+
 | pathFileName        | varchar(255) |
 +---------------------+--------------+

* Form:

  * Name: UploadAdvanced1
  * Table: Person

* FormElements

  * Name: name

    * Type: text
    * Label: Name

  * Name: mypathFileNamePicture

    * Type: upload
    * Label: Picture
    * Value: {{SELECT pathFileName FROM Note WHERE id={{slaveId}} }}
    * Parameter::

        fileDestination=fileadmin/user/{{id:R0}}-picture-{{filename}}
        slaveId={{SELECT id FROM Note WHERE pId={{id:R0}} AND type='picture' LIMIT 1}}
        sqlInsert={{INSERT INTO Note (pathFileName, type, pId) VALUE ('{{fileDestination}}', 'picture', {{id:R0}}) }}
        sqlUpdate={{UPDATE Note SET pathFileName = '{{fileDestination}}' WHERE id={{slaveId}} LIMIT 1}}
        sqlDelete={{DELETE FROM Note WHERE id={{slaveId}}  LIMIT 1}}

  * Name: mypathFileNameAvatar

    * Type: upload
    * Label: Avatar
    * Value: {{SELECT pathFileName FROM Note WHERE id={{slaveId}} }}
    * Parameter::

        fileDestination=fileadmin/user/{{id:R0}}-avatar-{{filename}}
        slaveId={{SELECT id FROM Note WHERE pId={{id:R0}} AND type='avatar' LIMIT 1}}
        sqlInsert={{INSERT INTO Note (pathFileName, type, pId) VALUE ('{{fileDestination}}', 'avatar', {{id:R0}}) }}
        sqlUpdate={{UPDATE Note SET pathFileName = '{{fileDestination}}' WHERE id={{slaveId}} LIMIT 1}}
        sqlDelete={{DELETE FROM Note WHERE id={{slaveId}}  LIMIT 1}}

Upload Form Advanced 2
^^^^^^^^^^^^^^^^^^^^^^

Table: Person

 +---------------------+--------------+
 | Name                | Type         |
 +=====================+==============+
 | id                  | int          |
 +---------------------+--------------+
 | name                | varchar(255) |
 +---------------------+--------------+
 | noteIdPicture       | int          |
 +---------------------+--------------+
 | noteIdAvatar        | int          |
 +---------------------+--------------+

Table: Note

 +---------------------+--------------+
 | Name                | Type         |
 +=====================+==============+
 | id                  | int          |
 +---------------------+--------------+
 | pathFileName        | varchar(255) |
 +---------------------+--------------+

* Form:

  * Name: UploadAdvanced2
  * Table: Person

* FormElements

  * Name: name

    * Type: text
    * Label: Name

  * Name: mypathFileNamePicture

    * Type: upload
    * Label: Picture
    * Value: {{SELECT pathFileName FROM Note WHERE id={{slaveId}} }}
    * Parameter::

        fileDestination=fileadmin/user/{{id:R0}}-picture-{{filename}}
        slaveId={{SELECT id FROM Note WHERE id={{noteIdPicture}} LIMIT 1}}
        sqlInsert={{INSERT INTO Note (pathFileName) VALUE ('{{fileDestination}}') }}
        sqlUpdate={{UPDATE Note SET pathFileName = '{{fileDestination}}' WHERE id={{slaveId}} LIMIT 1}}
        sqlDelete={{DELETE FROM Note WHERE id={{slaveId}}  LIMIT 1}}
        sqlAfter={{UPDATE Person SET noteIdPicture={{slaveId}} WHERE id={{id:R0}} LIMIT 1

  * Name: mypathFileNameAvatar

    * Type: upload
    * Label: Avatar
    * Value: {{SELECT pathFileName FROM Note WHERE id={{slaveId}} }}
    * Parameter::

        fileDestination=fileadmin/user/{{id:R0}}-avatar-{{filename}}
        slaveId={{SELECT id FROM Note WHERE id={{noteIdAvatar}} LIMIT 1}}
        sqlInsert={{INSERT INTO Note (pathFileName) VALUE ('{{fileDestination}}') }}
        sqlUpdate={{UPDATE Note SET pathFileName = '{{fileDestination}}' WHERE id={{slaveId}} LIMIT 1}}
        sqlDelete={{DELETE FROM Note WHERE id={{slaveId}}  LIMIT 1}}
        sqlAfter={{UPDATE Person SET noteIdAvatar={{slaveId}} WHERE id={{id:R0}} LIMIT 1

Typeahead: SQL
^^^^^^^^^^^^^^

Table: Person

 +---------------------+--------------+
 | Name                | Type         |
 +=====================+==============+
 | id                  | int          |
 +---------------------+--------------+
 | name                | varchar(255) |
 +---------------------+--------------+

* Form:

  * Name: PersonNameTypeahead
  * Table: Person

* FormElements

  * Name: name

    * Type: text
    * Label: Name
    * Parameter::

       typeAheadSql = SELECT name FROM Person WHERE name LIKE ? OR firstName LIKE ? LIMIT 100

Typeahead: LDAP with additional values
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Table: Person

 +---------------------+--------------+
 | Name                | Type         |
 +=====================+==============+
 | id                  | int          |
 +---------------------+--------------+
 | name                | varchar(255) |
 +---------------------+--------------+
 | firstname           | varchar(255) |
 +---------------------+--------------+
 | email               | varchar(255) |
 +---------------------+--------------+

* Form:

  * Name: PersonNameTypeaheadSetNames
  * Table: Person
  * Parameter::

      ldapServer = directory.example.com
      ldapBaseDn = ou=Addressbook,dc=example,dc=com

* FormElements

  * Name: email

    * Class: native
    * Type: text
    * Label: Email
    * Note: Name: {{cn:LE}}<br>Email: {{mail:LE}}
    * dynamicUpdate: checked
    * Parameter::

       # Typeahead
       typeAheadLdapSearch = (|(cn=*?*)(mail=*?*))
       typeAheadLdapValuePrintf	‘%s / %s’, cn, email
       typeAheadLdapIdPrintf	‘%s’, email

       # dynamicUpdate: show note
       fillStoreLdap
       ldapSearch = (mail={{email::alnumx}})
       ldapAttributes = cn, email

  * Name: fillLdapValues

    * Class: action
    * Type: afterSave
    * Parameter::

       fillStoreLdap
       ldapSearch = (mail={{email::alnumx}})
       ldapAttributes = cn, email

       slaveId={{id:R0}}
       sqlUpdate={{ UPDATE Person AS p SET p.name='{{cn:L:alnumx:s}}' WHERE p.id={{slaveId}} LIMIT 1 }}

.. _`import-merge-form`:

Import/merge form
-----------------

The form `copyFormFromExt` copies a form from table `ExtForm / ExtFormElement` to `Form / FormElement`. The import/merge
form:

* offers a drop down list with all forms of `ExtForm`,
* an input element for the new form name,
* create new Form.id
* copied FormElements get the new Form.id.
* the copied form will be opened in the FormEditor.

Installation:

* Play (do all sql statements on your QFQ database, e.g. via `mysql <dbname> < copyFormFromExt.sql` or `phpMyAdmin`) the
  file  *<ext_dir>/qfq/sql/copyFormFromExt.sql*.
* Insert a link/button 'Copy form from ExtForm' to open the import/merge form. A good place is the list of all forms (see `form-editor`_).
  E.g.: ::

    10.head = {{'b|p:id={{pageAlias:T}}&form=copyFormFromExt|t:Copy form from ExtForm' AS _link }} ...

If there are several T3/QFQ instances and if forms should be imported frequently/easily, set up a one shot
'import Forms from db xyz' like: ::

  10.sql = CREATE OR REPLACE table ExtForm SELECT * FROM <db xyz>.Form
  20.sql = CREATE OR REPLACE table ExtFormElement SELECT * FROM <db xyz>.FormElement


.. _`report`:

Report
======

QFQ content element
-------------------

The QFQ extension is activated through tt-content records. One or more tt-content records per page are necessary to render
*forms* and *reports*. Specify column and language per content record as wished.

The title of the QFQ content element will not be rendered. It's only visible in the backend for orientation of the webmaster.

To display a report on any given TYPO3 page, create a content element of type 'QFQ Element' (plugin) on that page.

A simple example
^^^^^^^^^^^^^^^^

Assume that the database has a table person with columns firstName and lastName. To create a simple list of all persons, we can do the following:
::

    10.sql = SELECT id AS pId, CONCAT(firstName, " ", lastName, " ") AS name FROM person

The '10' indicates a *root level* of the report (see section `Structure`_). The expresssion '10.sql' defines a SQL query
for the specific level. When the query is executed, it will return a result having one single column name containing first and last name
separated by a space character.

The HTML output, displayed on the page, resulting from only this definition, could look as follows:
::

    John DoeJane MillerFrank Star

..

I.e., QFQ will simply output the content of the SQL result row after row for each single level.

However, we can modify (wrap) the output by setting the values of various keys for each level: 10.rsep=<br/> for example
tells QFQ to separate the rows of the result by a HTML-line break. The final result in this case is:

::

    10.sql = SELECT id AS personId, CONCAT(firstName, " ", lastName, " ") AS name FROM person
    10.sep = <br>

HTML output:
::

    John Doe<br>Jane Miller<br>Frank Star

..

.. _`syntax-of-report`:

Syntax of *report*
------------------

All **root level queries** will be fired in the order specified by 'level' (Integer value).

For **each** row of a query (this means *all* queries), all subqueries will be fired once.

* E.g. if the outer query selects 5 rows, and a nested query select 3 rows, than the total number of rows are 5 x 3 = 15 rows.

There is a set of **variables** that will get replaced before the SQL-Query gets executed:

``{{<name>[:<store/s>[:...]]}}``
  Variables from specific stores.

``{{<name>:R}}`` - use case of the above generic definition. See also `access-column-values`_.

``{{<level>.<columnName>}}``
  Similar to  ``{{<name>:R}}`` but more specific. There is no sanitize class, escape mode or default value.

``{{<level>.line.count}}`` - Current row index
  This variable is specific, as it will be replaced before the query is fired in case of ``<level>`` is an outer/previous
  level or it will be replaced after a query is fired in case ``<level>`` is the current level.

``{{<level>.line.total}}``
  Total rows (MySQL ``num_rows`` for *SELECT* and *SHOW*, MySQL ``affected_rows`` for *UPDATE* and *INSERT*.

``{{<level>.line.insertId}}``
  Last insert id for *INSERT*.

``{{<level>.line.content}}``
  If the content of `<level>` have been stored, e.g. `<level>.content=hide`.


See :ref:`variables` for a full list of all available variables.

Different types of SQL queries are possible: SELECT, INSERT, UPDATE, DELETE, SHOW, REPLACE

Only SELECT and SHOW queries will fire subqueries.

Processing of the resulting rows and columns:

	* In general, all columns of all rows will be printed out sequentially.
	* On a per column base, printing of columns can be suppressed by starting the column name with an underscore '_'. E.g.
	  `SELECT id AS _id`.

     This might be useful to store values, which will be used later on in another query via the `{{id:R}}` or
     `{{<level>.columnName}}` variable. To suppress printing of a column, use a underscore as column name prefix. E.g.
     `SELECT id AS _id`

*Reserved column names* have a special meaning and will be processed in a special way. See
`Processing of columns in the SQL result`_ for details.

There are extensive ways to wrap columns and rows. See :ref:`wrapping-rows-and-columns`

Debug the bodytext
------------------
The parsed bodytext could be displayed by activating 'showDebugInfo' (:ref:`debug`) and specifying

::

    debugShowBodyText = 1

A small symbol with a tooltip will be shown, where the content record will be displayed on the webpage.
Note: :ref:`debug` information will only be shown with *showDebugInfo: yes* in configuration_.


.. _`inline-report`:

Inline Report editing
---------------------

For quick changes it can be bothersome to go to the TYPO3 backend to update the page content and reload the page.
For this reason, QFQ offers an inline report editing feature whenever there is a TYPO3 BE user logged in. A small
link symbol will appear on the right-hand side of each report record. Please note that the TYPO3 Frontend cache
is also deleted upon each inline report save.

In order for the inline report editing to work, QFQ needs to be able to access the T3 database. By default this database
is assumed to be accessible with the same credentials as specified with indexData and is assumed to be named similarly to
the indexData db name, but ending in _t3 instead of _db (e.g., mydb_db and mydb_t3).
For a standard installation and db setup, this should be the case.

You can however specify a custom T3 db name in config-qfq-php_:

::

    T3_DB_NAME = customT3DbName


Structure
---------

A report can be divided into several levels. This can make report definitions more readable because it allows for
splitting of otherwise excessively long SQL queries. For example, if your SQL query on the root level selects a number
of person records from your person table, you can use the SQL query on the second level to look up the city where each person lives.

See the example below:

::

    10.sql = SELECT id AS _pId, CONCAT(firstName, " ", lastName, " ") AS name FROM person
    10.rsep = <br>

    10.10.sql = SELECT CONCAT(postal_code, " ", city) FROM address WHERE pId = {{10.pId}}
    10.10.rbeg = (
    10.10.rend = )

..

This would result in

::

    John Doe (3004 Bern)
    Jane Miller (8008 Zürich)
    Frank Star (3012 Bern)

..

Text across several lines
^^^^^^^^^^^^^^^^^^^^^^^^^

To get better human readable SQL queries, it's possible to split a line across several lines. Lines
with keywords are on their own (`QFQ Keywords (Bodytext)`_ start a new line). If a line is not a 'keyword' line, it will
be appended to the last keyword line. 'Keyword' lines are detected on:

 * <level>.<keyword> =
 * {
 * <level>[.<sub level] {

Example::

    10.sql = SELECT 'hello world'
               FROM mastertable
    10.tail = End

    20.sql = SELECT 'a warm welcome'
               'some additional', 'columns'
               FROM another_table
               WHERE id>100

    20.head = <h3>
    20.tail = </h3>

Join mode: SQL
''''''''''''''

This is the default. All lines are joined with a *space* in between. E.g.: ::

    10.sql = SELECT 'hello world'
               FROM mastertable

Results to: `10.sql = SELECT 'hello world' FROM mastertable`

Notice the space between "...world'" and "FROM ...".

Join mode: strip whitespace
'''''''''''''''''''''''''''

Ending a line with a '\\' strips all leading and trailing whitespaces of that line joins the line directly (no extra
space in between). E.g.: ::

    10.sql = SELECT 'hello world', 'd:final.pdf \
                                    |p:id=export  \
                                    |t:Download' AS _pdf \

Results to: `10.sql = SELECT 'hello world', 'd:final.pdf|p:id=export|t:Download' AS _pdf`

Note: the '\\' does not force the joining, it only removes the whitespaces.

To get the same result, the following is also possible: ::

    10.sql = SELECT 'hello world', CONCAT('d:final.pdf'
                                    '|p:id=export',
                                    '|t:Download') AS _pdf

Nesting of levels
^^^^^^^^^^^^^^^^^

Levels can be nested. E.g.: ::

  10 {
    sql = SELECT ...
    5 {
        sql = SELECT ...
        head = ...
    }
  }

This is equal to: ::

  10.sql = SELECT ...
  10.5.sql = SELECT ...
  10.5.head = ...

Leading / trailing spaces
^^^^^^^^^^^^^^^^^^^^^^^^^

By default, leading or trailing whitespaces are removed from strings behind '='. E.g. 'rend =  test ' becomes 'test' for
rend. To prevent any leading or trailing spaces, surround them by using single or double ticks. Example: ::

	10.sql = SELECT name FROM Person
	10.rsep = ' '
	10.head = "Names: "


Braces character for nesting
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default, curly braces '{}' are used for nesting. Alternatively angle braces '<>', round braces '()' or square
braces '[]' are also possible. To define the braces to use, the **first line** of the bodytext has to be a comment line and the
last character of that line must be one of '{[(<'. The corresponding braces are used for that QFQ record. E.g.: ::

    # Specific code. >
    10 <
      sql = SELECT
      head = <script>
             data = [
               {
                 10, 20
               }
             ]
             </script>
    >


Per QFQ tt-content record, only one type of nesting braces can be used.

Be careful to:

* write nothing else than whitespaces/newline behind an **open brace**
* the **closing brace** has to be alone on a line. ::

   10.sql = SELECT 'Yearly Report'

   20 {
         sql = SELECT companyName FORM Company LIMIT 1
         head = <h1>
         tail = </h1>
   }

   30 {
         sql = SELECT depName FROM Department
         head = <p>
         tail = </p>
         5 {
               sql = SELECT 'detailed information for department'
               1.sql = SELECT name FROM Person LIMIT 7
               1.head = Employees:
         }
   }

   30.5.tail = More will follow

   50

   {
          sql = SELECT 'A query with braces on their own'
   }

.. _`access-column-values`:

Access column values
^^^^^^^^^^^^^^^^^^^^

Columns of the upper / outer level result can be accessed via variables in two ways

 * STORE_RECORD: `{{pId:R}}`
 * Level Key: `{{10.pId}}`

The STORE_RECORD will always be merged with previous content. The Level Keys are unique.

Multiple columns, with the same column name, can't be accessed individually. Only the last column is available.

Retrieving the *final* value of `special-column-names`_ is possible via '{{&<column>:R}}. Example: ::

  10.sql = SELECT 'p:home&form=Person|s|b:success|t:Edit' AS _link
  10.20.sql = SELECT '{{link:R}}', '{{&link:R}}'

The first column of row `10.20` returns 'p:home&form=Person|s|b:success|t:Edit',the second column returns
'<span class="btn btn-success"><a href="?home&s=badcaffee1234">Edit</a></span>'.

Example STORE_RECORD: ::

  10.sql= SELECT p.id AS _pId, p.name FROM Person AS p
  10.5.sql = SELECT adr.city, 'dummy' AS _pId FROM Address AS adr WHERE adr.pId={{pId:R}}
  10.5.20.sql = SELECT '{{pId:R}}'
  10.10.sql = SELECT '{{pId:R}}'

The line '10.10' will output 'dummy' in cases where there is at least one corresponding address.
If there are no addresses (all persons) it reports the person id.
If there is at least one address, it reports 'dummy', cause that's the last stored content.

Example 'level': ::

  10.sql= SELECT p.id AS _pId, p.name FROM Person AS p
  10.5.sql = SELECT adr.city, 'dummy' AS _pId FROM Address AS adr WHERE adr.pId={{10.pId}}
  10.5.20.sql = SELECT '{{10.pId}}'
  10.10.sql = SELECT '{{10.pId}}'


Notes to the level:

+-------------+------------------------------------------------------------------------------------------------------------------------+
| Levels      |A report is divided into levels. The Example has levels *10*, *20*, *30*, *30.5*, *30.5.1*, *50*                        |
+-------------+------------------------------------------------------------------------------------------------------------------------+
| Qualifier   |A level is divided into qualifiers *30.5.1* has 3 qualifiers *30*, *5*, *1*                                             |
+-------------+------------------------------------------------------------------------------------------------------------------------+
| Root levels |Is a level with one qualifier. E.g.: 10                                                                                 |
+-------------+------------------------------------------------------------------------------------------------------------------------+
| Sub levels  |Is a level with more than one qualifier. E.g. levels *30.5* or *30.5.1*                                                 |
+-------------+------------------------------------------------------------------------------------------------------------------------+
| Child       |The level *30* has one child and child child: *30.5* and *30.5.1*                                                       |
+-------------+------------------------------------------------------------------------------------------------------------------------+
| Example     | *10*, *20*, *30*, *50** are root level and will be completely processed one after each other.                          |
|             | *30.5* will be executed as many times as *30* has row numbers.                                                         |
|             | *30.5.1*  will be executed as many times as *30.5* has row numbers.                                                    |
+-------------+------------------------------------------------------------------------------------------------------------------------+

Report Example 1: ::

    # Displays current date
    10.sql = SELECT CURDATE()

    # Show all students from the person table
    20.sql = SELECT p.id AS pId, p.firstName, " - ", p.lastName FROM person AS p WHERE p.typ LIKE "student"

    # Show all the marks from the current student ordered chronological
    20.25.sql = SELECT e.mark FROM exam AS e WHERE e.pId={{20.pId}} ORDER BY e.date

    # This query will never be fired, cause there is no direct parent called 20.30.
    20.30.10.sql = SELECT 'never fired'

.. _wrapping-rows-and-columns:

Wrapping rows and columns: Level
--------------------------------

Order and nesting of queries, will be defined with a typoscript-like syntax: level.sublevel1.subsublevel2. ...
Each 'level' directive needs a final key, e.g: 20.30.10. **sql**. A key **sql** is necessary in order to process a level.
See all `QFQ Keywords (Bodytext)`_.

Processing of columns in the SQL result
---------------------------------------

* The content of all columns of all rows will be printed sequentially, without separator.
* Rows with `special-column-names`_  will be processed in a special way.

.. _special-column-names:

Special column names
--------------------

* Special column names always start with '_'.
* Column names, which start with a '_' and which are not reserved (=special column name), will not be printed. Nevertheless,
  access to it via the {{<level>.<column>}} variable (without '_') are possible.
* The input parameters for the processing function are stored as column values.
* Single parameters are delimited by the '|' character.
* Parameters are identified by the function either

  * by their **order**
  * or by a **one character qualifier** followed by the ':' character, placed in front of the actual parameter value.

+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Reserved column name   | Purpose                                                                                                                                                                                     |
+========================+=============================================================================================================================================================================================+
| _link                  |Easily create links with different features.                                                                                                                                                 |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _mailto                |Quickly create email links. A click on the link will open the default mailer. The address is encrypted via JS against email bots.                                                            |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _pageX or _PageX       |Shortcut version of the link interface for fast creation of internal links. The column name is composed of the string *page*/*Page* and a optional character to specify the type of the link.|
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _pdf, _file, _zip      |Shortcut version of the link interface for fast creation of `download`_ links. Used to offer single file download or to concatenate several PDFs and printout of websites to one PDF file.   |
| _Pdf, _File, _Zip      |                                                                                                                                                                                             |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _yank                  | `copyToClipboard`_. Shortcut version of the link interface                                                                                                                                  |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _sendmail              |Send emails.                                                                                                                                                                                 |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _exec                  |Run batch files or executables on the webserver.                                                                                                                                             |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _vertical              |Render Text vertically. This is useful for tables with limited column width.                                                                                                                 |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _img                   |Display images.                                                                                                                                                                              |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _bullet                |Display a blue/gray/green/pink/red/yellow bullet. If none color specified, show nothing.                                                                                                     |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _check                 |Display a blue/gray/green/pink/red/yellow checked sign. If none color specified, show nothing.                                                                                               |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _nl2br                 |All newline characters will be converted to `<br>`.                                                                                                                                          |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _striptags             |HTML Tags will be stripped.                                                                                                                                                                  |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _htmlentities          |Characters will be encoded to their HTML entity representation.                                                                                                                              |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _fileSize              |Show file size of a given file                                                                                                                                                               |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _mimeType              |Show mime type of a given file                                                                                                                                                               |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _thumbnail             |Create thumbnails on the fly. See `column-thumbnail`_.                                                                                                                                       |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _monitor               |Constantly display a file. See `column-monitor`_.                                                                                                                                            |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _XLS                   |Used for Excel export. Append a `newline` character at the end of the string. Token must be part of string. See `excel-export`_.                                                             |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _XLSs                  |Used for Excel export. Prepend the token 's=' and append a `newline` character at the string. See `excel-export`_.                                                                           |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _XLSb                  |Used for Excel export. Like '_XLSs' but encode the string base64. See `excel-export`_.                                                                                                       |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _XLSn                  |Used for Excel export. Prepend 'n=' and append a `newline` character around the string. See `excel-export`_.                                                                                 |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _+html-tag attributes  |The content will be wrapped with '<html-tag attributes>'. Example: SELECT 'example' AS '_+a href="http://example.com"' creates '<a href="http://example.com">example</a>'                    |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| _=varname              |The content will be saved in store 'user' under 'varname'. Retrieve it later via {{varname:U}}. See `STORE_USER`_, `store_user_examples`_                                                    |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_<nonReservedName>      |Suppress output. Column names with leading underscore are used to select data from the database and make it available in other parts of the report without generating any output.            |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

.. _column-link:

Column: _link
^^^^^^^^^^^^^

* Most URLs will be rendered via class link.
* Column names like `_pagee`, `_mailto`, ... are wrapper to class link.
* The parameters for link contains a prefix to make them position-independent.

+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|URL|IMG|Meaning       |Qualifier                          |Example                    |Description                                                                                                                             |
+===+===+==============+===================================+===========================+========================================================================================================================================+
|x  |   |URL           |u:<url>                            |u:http://www.example.com   |If an image is specified, it will be rendered inside the link, default link class: external                                             |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|x  |   |Mail          |m:<email>                          |m:info@example.com         |Default link class: email                                                                                                               |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|x  |   |Page          |p:<pageId>                         |p:impressum                |Prepend '?' or '?id=', no hostname qualifier (automatically set by browser), default value: {{pageId}}                                  |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|x  |   |Download      |d:[<exportFilename>]               |d:complete.pdf             |Link points to `api/download.php`. Additional parameter are encoded into a SIP. 'Download' needs an enabled SIP.  See `download`_.      |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|x  |   |Copy to       |y:[some content]                   |y:this will be copied      |Click on it copies the value of 'y:' to the clipboard. Optional a file ('F:...') might be specified as source.                          |
|   |   |clipboard     |                                   |                           |See `copyToClipboard`_.                                                                                                                 |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Text          |t:<text>                           |t:Firstname Lastname       |-                                                                                                                                       |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Render        |r:<mode>                           |r:3                        |See: `render-mode`_, Default: 0                                                                                                         |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Button        | b[:0|1|<btn class>]               | b:0, b:1, b:success       |'b', 'b:1': a bootstrap button is created. 'b:0' disable the button. <btn class>: default, primary, success, info, warning,danger       |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Picture       |P:<filename>                       |P:bullet-red.gif           |Picture '<img src="bullet-red.gif"alt="....">'.                                                                                         |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Edit          |E                                  |E                          |Show 'edit' icon as image                                                                                                               |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |New           |N                                  |N                          |Show 'new' icon as image                                                                                                                |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Delete        |D                                  |D                          |Show 'delete' icon as image (only the icon, no database record 'delete' functionality)                                                  |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Help          |H                                  |H                          |Show 'help' icon as image                                                                                                               |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Info          |I                                  |I                          |Show 'information' icon as image                                                                                                        |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Show          |S                                  |S                          |Show 'show' icon as image                                                                                                               |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Glyph         |G:<glyphname>                      |G:glyphicon-envelope       |Show <glyphname>. Check: http://getbootstrap.com/components/                                                                            |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Bullet        |B:[<color>]                        |B:green                    |Show bullet with '<color>'. Colors: blue, gray, green, pink, red, yellow. Default Color: green.                                         |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Check         |C:[<color>]                        |C:green                    |Show checked with '<color>'. Colors: blue, gray, green, pink, red, yellow. Default Color: green.                                        |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Thumbnail     |T:<pathFilename>                   |T:fileadmin/file.pdf       |Creates a thumbnail on the fly. Size is specified via 'W'. See `column-thumbnail`_                                                      |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Dimension     |W:[width]x[height]                 |W:50x , W:x60 , W:50x60    |Defines the pixel size of thumbnail.  See `column-thumbnail`_                                                                           |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |URL Params    |U:<key1>=<value1>[&<keyN>=<valueN>]|U:a=value1&b=value2&c=...  |Any number of additional Params. Links to forms: U:form=Person&r=1234. Used to create 'record delete'-links.                            |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Tooltip       |o:<text>                           |o:More information here    |Tooltip text                                                                                                                            |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Alttext       |a:<text>                           |a:Name of person           |a) Alttext for images, b) Message text for `download`_ popup window.                                                                    |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Class         |c:[n|<text>]                       |c:text-muted               |CSS class for link. n:no class attribute, <text>: explicit named                                                                        |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Target        |g:<text>                           |g:_blank                   |target=_blank,_self,_parent,<custom>. Default: no target                                                                                |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Question      |q:<text>                           |q:please confirm           |See: `question`_. Link will be executed only if user clicks ok/cancel, default: 'Please confirm'                                        |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Encryption    |e:0|1|...                          |e:1                        |Encryption of the e-mail: 0: no encryption, 1:via Javascript (default)                                                                  |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Right         |R                                  |R                          |Defines picture position: Default is 'left' (no definition) of the 'text'. 'R' means 'right' of the 'text'                              |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |SIP           |s[:0|1]                            |s, s:0, s:1                |If 's' or 's:1' a SIP entry is generated with all non Typo 3 Parameters. The URL contains only parameter 's' and Typo 3 parameter       |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Mode          |M:file|pdf|zip                     |M:file, M:pdf, M:zip       |Mode. Used to specify type of download. One or more element sources needs to be configured. See `download`_.                            |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |File          |F:<filename>                       |F:fileadmin/file.pdf       |Element source for download mode file|pdf|zip. See `download`_.                                                                         |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Delete record | x[:a|r|c]                         |x, x:r, x:c                |a: ajax (only QFQ internal used), r: report (default), c: close (current page, open last page)                                          |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+

.. _render-mode:

Render mode
^^^^^^^^^^^

The following table might be hard to read - but it's really useful to understand. It solves a lot of different situations.
If there are no special condition (like missing value, or suppressed links), render mode 0 is sufficient.
But if the URL text is missing, or the URL is missing, OR the link should be rendered in sql row 1-10, but not 5, then
render mode might dynamically control the rendered link.

* Column *Mode* is the render mode and controls how the link is rendered.

+------------+---------------------+--------------------+------------------+---------------------------------------------------------------------------+
|Mode        |Given: url & text    |Given: only url     | Given: only text |Description                                                                |
+============+=====================+====================+==================+===========================================================================+
|0 (default) |<a href=url>text</a> |<a href=url>url</a> |                  |text or image will be shown, only if there is a url, page or mailto        |
+------------+---------------------+--------------------+------------------+---------------------------------------------------------------------------+
|1           |<a href=url>text</a> |<a href=url>url</a> |text              |text or image will be shown, independently of whether there is a url or not|
+------------+---------------------+--------------------+------------------+---------------------------------------------------------------------------+
|2           |<a href=url>text</a> |                    |                  |no link if text is empty                                                   |
+------------+---------------------+--------------------+------------------+---------------------------------------------------------------------------+
|3           |text                 |url                 |text              |no link, only text or image, incl. Optional tooltip. For Bootstrap buttons |
|            |                     |                    |                  | r:3 will set the button to disable and no link/sip is rendered.           |
+------------+---------------------+--------------------+------------------+---------------------------------------------------------------------------+
|4           |url                  |url                 |text              |no link, show text, if text is empty, show url, incl. optional tooltip     |
+------------+---------------------+--------------------+------------------+---------------------------------------------------------------------------+
|5           |                     |                    |                  |nothing at all                                                             |
+------------+---------------------+--------------------+------------------+---------------------------------------------------------------------------+
|6           | pure text           |                    |pure text         |no link, pure text                                                         |
+------------+---------------------+--------------------+------------------+---------------------------------------------------------------------------+
|7           | pure url            |pure url            |                  |no link, pure url                                                          |
+------------+---------------------+--------------------+------------------+---------------------------------------------------------------------------+

::

    10.sql = SELECT CONCAT('u:', p.homepage, IF(p.showHomepage='yes', '|r:0', '|r:5') ) AS _link FROM Person AS p

Tip:

An easy way to switch between different options of rendering a link, incl. Bootstrap buttons, is to use the render mode.

* no render mode or 'r:0' - the full functional link/button.
* 'r:3' - the link/button is rendered with text/image/glyph/tooltip ... but without a HTML a-tag! For Bootstrap button, the button get the 'disabled' class.
* 'r:5' - no link/button at all.

Link Examples
^^^^^^^^^^^^^

+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
|SQL-Query                                                              | Result                                                                                                                                  |
+=======================================================================+=========================================================================================================================================+
| SELECT "m:info@example.com" AS _link                                  | info@example.com as linked text, encrypted with javascript, class=external                                                              |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "m:info@example.com|c:0" AS _link                              | info@example.com as linked text, not encrypted, class=external                                                                          |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "m:info@example.com|P:mail.gif" AS _link                       | info@example.com as linked image mail.gif, encrypted with javascript, class=external                                                    |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "m:info@example.com|P:mail.gif|o:Email" AS _link               | *info@example.com* as linked image mail.gif, encrypted with javascript, class=external, tooltip: "sendmail"                             |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "m:info@example.com|t:mailto:info@example.com|o:Email" AS link | 'mail to *info@example.com*' as linked text, encrypted with javascript, class=external                                                  |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "u:www.example.com" AS _link                                   | www.example as link, class=external                                                                                                     |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "u:http://www.example.com" AS _link                            | *http://www.example* as link, class=external                                                                                            |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "u:www.example.com|q:Please confirm" AS _link                  | www.example as link, class=external, See: `question`_                                                                                   |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "u:www.example.com|c:nicelink" AS _link                        | *http://www.example* as link, class=nicelink                                                                                            |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "p:form_person&note=Text|t:Person" AS _link                    | <a href="?form_person&note=Text">Person</a>                                                                                             |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "p:form_person|E" AS _link                                     | <a href="?form_person"><img alttext="Edit" src="typo3conf/ext/qfq/Resources/Public/icons/edit.gif"></a>                                 |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "p:form_person|E|g:_blank" AS _link                            | <a target="_blank" href="?form_person"><img alttext="Edit" src="typo3conf/ext/qfq/Resources/Public/icons/edit.gif"></a>                 |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "p:form_person|C" AS _link                                     | <a href="?form_person"><img alttext="Check" src="typo3conf/ext/qfq/Resources/Public/icons/checked-green.gif"></a>                       |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "p:form_person|C:green" AS _link                               | <a href="?form_person"><img alttext="Check" src="typo3conf/ext/qfq/Resources/Public/icons/checked-green.gif"></a>                       |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "U:form=Person&r=123|x|D" as _link                             | <a href="typo3conf/ext/qfq/qfq/api/delete.php?s=badcaffee1234"><span class="glyphicon glyphicon-trash" ></span>"></a>                   |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "U:form=Person&r=123|x|t:Delete" as _link                      | <a href="typo3conf/ext/qfq/qfq/api/delete.php?s=badcaffee1234">Delete</a>                                                               |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT "s:1|d:full.pdf|M:pdf|p:id=det1&r=12|p:id=det2|F:cv.pdf|       | <a href="typo3conf/ext/qfq/qfq/api/download.php?s=badcaffee1234">Download</a>                                                           |
|         t:Download|a:Create complete PDF - please wait" as _link      |                                                                                                                                         |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT  "y:iatae3Ieem0jeet|t:Password|o:Clipboard|b" AS _link         | <button class="btn btn-info" onClick="new QfqNS.Clipboard({text: 'iatae3Ieem0jeet'});" title='Copy to clipboard'>Password</button>      |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+
| SELECT  "y|s:1|F:dir/data.R|t:Data|o:Clipboard|b" AS _link            | <button class="btn btn-info" onClick="new QfqNS.Clipboard({uri: 'typo3conf/.../download.php?s=badcaffee1234'});"                        |
|                                                                       | title='Copy to clipboard'>Data</button>                                                                                                 |
+-----------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------+

.. _question:

Question
^^^^^^^^

**Syntax**

::

    q[:<alert text>[:<level>[:<positive button text>[:<negative button text>[:<timeout>[:<flag modal>]]]]]]


* If a user clicks on a link, an alert is shown. If the user answers the alert by clicking on the 'positive button', the browser opens the specified link.
  If the user click on the negative answer (or waits for timout), the alert is closed and the browser does nothing.
* All parameter are optional.
* Parameter are separated by ':'
* To use ':' inside the text, the colon has to be escaped by '\\'. E.g. 'ok\\: I understand'.

+----------------------+--------------------------------------------------------------------------------------------------------------------------+
|   Parameter          |   Description                                                                                                            |
+======================+==========================================================================================================================+
| Text                 | The text shown by the alert. HTML is allowed to format the text. Any ':' needs to be escaped. Default: 'Please confirm'. |
+----------------------+--------------------------------------------------------------------------------------------------------------------------+
| Level                | success, info, warning, danger                                                                                           |
+----------------------+--------------------------------------------------------------------------------------------------------------------------+
| Positive button text | Default: 'Ok'                                                                                                            |
+----------------------+--------------------------------------------------------------------------------------------------------------------------+
| Negative button text | Default: 'Cancel'                                                                                                        |
+----------------------+--------------------------------------------------------------------------------------------------------------------------+
| Timeout in seconds   | 0: no timeout, >0: after the specified time in seconds, the alert will dissapear and behaves like 'negative answer'      |
+----------------------+--------------------------------------------------------------------------------------------------------------------------+
| Flag modal           | 0: Alert behaves not modal. 1: (default) Alert behaves modal.                                                            |
+----------------------+--------------------------------------------------------------------------------------------------------------------------+

Examples:

+------------------------------------------------------------+---------------------------------------------------------------------------+
|   SQL-Query                                                |   Result                                                                  |
+============================================================+===========================================================================+
| SELECT "p:form_person|q:Edit Person:warn" AS _link         | Shows alert with level 'warn'                                             |
+------------------------------------------------------------+---------------------------------------------------------------------------+
| SELECT "p:form_person|q:Edit Person::I do:No way" AS _link | Instead of 'Ok' and 'Cancel', the button text will be 'I do' and 'No way' |
+------------------------------------------------------------+---------------------------------------------------------------------------+
| SELECT "p:form_person|q:Edit Person:::10" AS _link         | The Alert will be shown 10 seconds                                        |
+------------------------------------------------------------+---------------------------------------------------------------------------+
| SELECT "p:form_person|q:Edit Person:::10:0" AS _link       | The Alert will be shown 10 seconds and is not modal.                      |
+------------------------------------------------------------+---------------------------------------------------------------------------+

.. _column_pageX:

Columns: _page[X]
^^^^^^^^^^^^^^^^^

The colum name is composed of the string *page* and a trailing character to specify the type of the link.


**Syntax**

::

    10.sql = SELECT "[options]" AS _page[<link type>]

    with: [options] = [p:<page & param>][|t:<text>][|o:<tooltip>][|q:<question parameter>][|c:<class>][|g:<target>][|r:<render mode>]

    <link type> = c,d,e,h,i,n,s

..

+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|  column name  |  Purpose                                      |default value of question parameter  |  Mandatory parameters                        |
+===============+===============================================+=====================================+==============================================+
|_page          |Internal link without a grafic                 |empty                                |p:<pageId>[&param]                            |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|_pagec         |Internal link without a grafic, with question  |*Please confirm!*                    |p:<pageId>[&param]                            |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|_paged         |Internal link with delete icon (trash)         |*Delete record ?*                    | | U:form=<formname>&r=<record id> *or*       |
|               |                                               |                                     | | U:table=<tablename>&r=<record id>          |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|_pagee         |Internal link with edit icon (pencil)          |empty                                |p:<pageId>[&param]                            |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|_pageh         |Internal link with help icon (question mark)   |empty                                |p:<pageId>[&param]                            |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|_pagei         |Internal link with information icon (i)        |empty                                |p:<pageId>[&param]                            |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|_pagen         |Internal link with new icon (sheet)            |empty                                |p:<pageId>[&param]                            |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|_pages         |Internal link with show icon (magnifier)       |empty                                |p:<pageId>[&param]                            |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+


* All parameter are optional.
* Optional set of predefined icons.
* Optional set of dialog boxes.

+--------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|  Parameter   |  Description                                                                                    |  Default value                                           |Example                                                        |
+==============+=================================================================================================+==========================================================+===============================================================+
|<page>        |TYPO3 page id or page alias.                                                                     |The current page: *{{pageId}}*                            |45 application application&N_param1=1045                       |
+--------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<text>        |Text, wrapped by the link. If there is an icon, text will be displayed to the right of it.       |empty string                                              |                                                               |
+--------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<tooltip>     |Text to appear as a ToolTip                                                                      |empty string                                              |                                                               |
+--------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<question>    |If there is a question text given, an alert will be opened. Only if the user clicks on 'ok',     |**Expected "=" to follow "see"**                          |                                                               |
|              |the link will be called                                                                          |                                                          |                                                               |
+--------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<class>       |CSS Class for the <a> tag                                                                        |                                                          |                                                               |
|              |                                                                                                 |                                                          |                                                               |
+--------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<target>      |Parameter for HTML 'target='. F.e.: Opens a new window                                           |empty                                                     |P                                                              |
+--------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<render mode> |Show/render a link at all or not. See `render-mode`_                                             |                                                          |                                                               |
+--------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<create sip>  |s                                                                                                |                                                          |'s': create a SIP                                              |
+--------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+

.. _column_paged:

Column: _paged
^^^^^^^^^^^^^^

These column offers a link, with a confirmation question, to delete one record (mode 'table') or a bunch of records
(mode 'form'). After deleting the record(s), the current page will be reloaded in the browser.

**Syntax**

::

    10.sql = SELECT "U:table=<tablename>&r=<record id>|q:<question>|..." AS _paged
    10.sql = SELECT "U:form=<formname>&r=<record id>|q:<question>|..." AS _paged

..

If the record to delete contains column(s), whose column name match on `%pathFileName%` and such a
column points to a real existing file, such a file will be deleted too. If the table contains records where the specific
file is multiple times referenced, than the file is not deleted (it would break the still existing references). Multiple
references are not found, if they use different colummnnames or tablenames.

Mode: table
'''''''''''

* `table=<table name>`
* `r=<record id>`

Deletes the record with id '<record id>' from table '<table name>'.

Mode: form
''''''''''

* `form=<form name>`
* `r=<record id>`

Deletes the record with id '<record id>' from the table specified in form '<form name>' as primary table.
Additional action *FormElement* of type *beforeDelete* or *afterDelete* will be fired too.

Examples:
'''''''''

::

    10.sql = SELECT 'U:table=Person&r=123|q:Do you want delete John Doe?' AS _paged
    10.sql = SELECT 'U:form=person-main&r=123|q:Do you want delete John Doe?' AS _paged

.. _column_ppageX:

Columns: _Page[X]
^^^^^^^^^^^^^^^^^

* Similar to `_page[X]`
* Parameter are position dependent and therefore without a qualifier!

::

    "[<page id|alias>[&param=value&...]] | [text] | [tooltip] | [question parameter] | [class] | [target] | [render mode]" as _Pagee.

.. _column_ppaged:

Column: _Paged
^^^^^^^^^^^^^^

* Similar to `_paged`
* Parameter are position dependent and therefore without a qualifier!

::

    "[table=<table name>&r-<record id>[&param=value&...] | [text] | [tooltip] | [question parameter] | [class] | [render mode]" as _Paged.
    "[form=<form name>&r-<record id>[&param=value&...] | [text] | [tooltip] | [question parameter] | [class] | [render mode]" as _Paged.

.. _column_vertical:

Column: _vertical
^^^^^^^^^^^^^^^^^

Render text vertically. This is useful for tables with limited column width. The vertical rendering is achieved via CSS tranformations
(rotation) defined in the style attribute of the wrapping tag. You can optionally specify the rotation angle.

**Syntax**

::

    10.sql = SELECT "<text>|[<angle>]" AS _vertical

..

+-------------+-------------------------------------------------------------------------------------------------------+-----------------+
|**Parameter**|**Description**                                                                                        |**Default value**|
+=============+=======================================================================================================+=================+
|<text>       |The string that should be rendered vertically.                                                         |none             |
+-------------+-------------------------------------------------------------------------------------------------------+-----------------+
|<angle>      |How many degrees should the text be rotated? The angle is measured clockwise from baseline of the text.|*270*            |
+-------------+-------------------------------------------------------------------------------------------------------+-----------------+

The text is surrounded by some HTML tags in an effort to make other elements position appropriately around it.
This works best for angles close to 270 or 90.

**Minimal Example**

::


    10.sql = SELECT "Hallo" AS _vertical
    20.sql = SELECT "Hallo|90" AS _vertical
    20.sql = SELECT "Hallo|-75" AS _vertical

..

.. _column_mailto:

Column: _mailto
^^^^^^^^^^^^^^^

Easily create Email links.

**Syntax**

::


    10.sql = SELECT "<email address>|[<link text>]" AS _mailto

..



+--------------+----------------------------------------------------------------------------------------+-------------+
|**Parameter** |**Description**                                                                         |**Default    |
|              |                                                                                        |value**      |
+==============+========================================================================================+=============+
|<emailaddress>| The email address where the link should point to.                                      |none         |
+--------------+----------------------------------------------------------------------------------------+-------------+
|<linktext>    | The text that should be displayed on the website and be linked to the email address.   |none         |
|              | This will typically be the name of the recipient. If this parameter is omitted,        |             |
|              | the email address will be displayed as link text.                                      |             |
+--------------+----------------------------------------------------------------------------------------+-------------+


**Minimal Example**

::


    10.sql = SELECT "john.doe@example.com" AS _mailto

..



**Advanced Example**

::


    10.sql = SELECT "john.doe@example.com|John Doe" AS _mailto

..

.. _column_sendmail:

Column: _sendmail
^^^^^^^^^^^^^^^^^

Format: ::

    t:<TO:email[,email]>|f:<FROM:email>|s:<subject>|b:<body>
        [|c:<CC:email[,email]]>[|B:<BCC:email[,email]]>[|r:<REPLY-TO:email>]
        [|A:<flag autosubmit: on/off>][|g:<grId>][|x:<xId>][|y:<xId2>][|z:<xId3>][|h:<mail header>]
        [|e:<subject encode: encode/decode/none>][E:<body encode: encode/decode/none>][|mode:html]
        [|C][d:<filename of the attachment>][|F:<file to attach>][|u:<url>][|p:<T3 uri>]

The following parameters can also be written as complete words for ease of use: ::

    to:<email[,email]>|from:<email>|subject:<subject>|body:<body>
        [|cc:<email[,email]]>[|bcc:<email[,email]]>[|reply-to:<email>]
        [|autosubmit:<on/off>][|grid:<grid>][|xid:<xId>][|xid2:<xId2>][|xid3:<xId3>][|header:<mail header>]
        [|mode:html]

Send emails. Every mail will be logged in the table `mailLog`. Attachments are supported.

**Syntax**

::

    10.sql = SELECT "t:john@doe.com|f:jane@doe.com|s:Reminder tomorrow|b:Please dont miss the meeting tomorrow" AS _sendmail
    10.sql = SELECT "t:john@doe.com|f:jane@doe.com|s:Reminder tomorrow|b:Please dont miss the meeting tomorrow|A:off|g:1|x:2|y:3|z:4" AS _sendmail

..

+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
|**Token**     | **Parameter**                          |**Description**                                                                                   |**Required**|
| short / long |                                        |                                                                                                  |            |
+==============+========================================+==================================================================================================+============+
| | f          | email                                  |**FROM**: Sender of the email. Optional: 'realname <john@doe.com>'                                |    yes     |
| | from       |                                        |                                                                                                  |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | t          | email[,email]                          |**TO**: Comma separated list of receiver email addresses. Optional: `realname <john@doe.com>`     |    yes     |
| | to         |                                        |                                                                                                  |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | c          | email[,email]                          |**CC**: Comma separated list of receiver email addresses. Optional: 'realname <john@doe.com>'     |            |
| | cc         |                                        |                                                                                                  |    yes     |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | B          | email[,email]                          |**BCC**: Comma separated list of receiver email addresses. Optional: 'realname <john@doe.com>'    |            |
| | bcc        |                                        |                                                                                                  |    yes     |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | r          | REPLY-TO:email                         |**Reply-to**: Email address to reply to (if different from sender)                                |            |
| | reply-to   |                                        |                                                                                                  |    yes     |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | s          | Subject                                |**Subject**: Subject of the email                                                                 |    yes     |
| | subject    |                                        |                                                                                                  |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | b          | Body                                   |**Body**: Message - see also: `html-formatting`_                                                  |    yes     |
| | body       |                                        |                                                                                                  |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | h          | Mail header                            |**Custom mail header**: Separate multiple header with \\r\\n                                      |            |
| | header     |                                        |                                                                                                  |    yes     |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| F            | Attach file                            |**Attachment**: File to attach to the mail. Repeatable.                                           |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| u            | Attach created PDF of a given URL      |**Attachment**: Convert the given URL to a PDF and attach it the mail. Repeatable.                |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| p            | Attach created PDF of a given T3 URL   |**Attachment**: Convert the given URL to a PDF and attach it the mail. Repeatable.                |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| d            | Filename of the attachment             |**Attachment**: Useful for URL to PDF converted attachments. Repeatable.                          |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| C            | Concat multiple F|p|u| together        |**Attachment**: All following (until the next 'C') 'F|p|u' concatenated to one attachment.        |            |
|              |                                        | Repeatable.                                                                                      |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | A          | flagAutoSubmit  'on' / 'off'           |If 'on' (default), add mail header 'Auto-Submitted: auto-send' - suppress OoO replies             |            |
| | autosubmit |                                        |                                                                                                  |    yes     |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | g          | grId                                   |Will be copied to the mailLog record. Helps to setup specific logfile queries                     |            |
| | grid       |                                        |                                                                                                  |    yes     |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | x          | xId                                    |Will be copied to the mailLog record. Helps to setup specific logfile queries                     |            |
| | xid        |                                        |                                                                                                  |    yes     |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | y          | xId2                                   |Will be copied to the mailLog record. Helps to setup specific logfile queries                     |            |
| | xid2       |                                        |                                                                                                  |    yes     |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| | z          | xId3                                   |Will be copied to the mailLog record. Helps to setup specific logfile queries                     |            |
| | xid3       |                                        |                                                                                                  |    yes     |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| e            | encode|decode|none                     |**Subject**: will be htmlspecialchar() encoded, decoded (default) or none (untouched)             |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| E            | encode|decode|none                     |**Body**: will be htmlspecialchar() encoded, decoded (default) or none (untouched).               |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+
| M            | html                                   |**Body**: will be send as a HTML mail.                                                            |            |
+--------------+----------------------------------------+--------------------------------------------------------------------------------------------------+------------+

* **e|E**: By default, QFQ stores values 'htmlspecialchars()' encoded. If such values have to send by email, the html entities are
  unwanted. Therefore the default setting for 'subject' und 'body' is to decode the values via 'htmlspecialchars_decode()'.
  If this is not wished, it can be turned off by `e=none` and/or `E=none`.


**Minimal Example**

::

    10.sql = SELECT "t:john.doe@example.com|f:company@example.com|s:Latest News|b:The new version is now available." AS _sendmail

..

This will send an email with subject *Latest News* from company@example.com to john.doe@example.com.

**Advanced Examples**

::

    10.sql = SELECT "t:customer1@example.com,Firstname Lastname <customer2@example.com>, Firstname Lastname <customer3@example.com>| \\
                     f:company@example.com|s:Latest News|b:The new version is now available.|r:sales@example.com|A:on|g:101|x:222|c:ceo@example.com|B:backup@example.com" AS _sendmail

..

This will send an email with subject *Latest News* from company@example.com to customer1, customer2 and customer3 by
using a realname for customer2 and customer3 and suppress generating of OoO answer if any receiver is on vacation.
Additional the CEO as well as backup will receive the mail via CC and BCC.

For debugging, please check `REDIRECT_ALL_MAIL_TO`_.

.. _html-formatting:

**Mail Body HTML Formatting**

In order to send an email with HTML formatting, such as bold text or bullet lists, specify 'mode=html'.
The subsequent contents will be interpreted as HTML and is rendered correctly by most email programs.

.. _attachment:

Attachment
''''''''''

The following options are provided to attach files to an email:

+-------+------------------------------------------------------+--------------------------------------------------------+
| Token | Example                                              | Comment                                                |
+=======+======================================================+========================================================+
| F     | F:fileadmin/file3.pdf                                | Single file  to attach                                 |
+-------+------------------------------------------------------+--------------------------------------------------------+
| u     | u:www.example.com/index.html?key=value&...           | A URL, will be converted to a PDF and than attached.   |
+-------+------------------------------------------------------+--------------------------------------------------------+
| p     | p:?id=export&r=123&_sip=1                            | A SIP protected local T3 page.                         |
|       |                                                      | Will be converted to a PDF and than attached.          |
+-------+------------------------------------------------------+--------------------------------------------------------+
| d     | d:myfile.pdf                                         | Name of the attachment in the email.                   |
+-------+------------------------------------------------------+--------------------------------------------------------+
| C     | C|u:http://www.example.com|F:file1.pdf|C|F:file2.pdf | Concatenate all named sources to one PDF file. The     |
|       |                                                      | souces has to be PDF files or a web page, which will be|
|       |                                                      | converted to a PDF first.                              |
+-------+------------------------------------------------------+--------------------------------------------------------+

Any combination (incl. repeating them) are possible. Any source will be added as a single attachment.

Optional any number of sources can be concatenated to a single PDF file: 'C|F:<file1>|F:<file2>|p:export&a=123'.

Examples in Report::

	# One file attached.
	10.sql = SELECT "t:john.doe@example.com|f:company@example.com|s:Latest News|b:The new version is now available.|F:fileadmin/summary.pdf" AS _sendmail

	# Two files attached.
	10.sql = SELECT "t:john.doe@example.com|f:company@example.com|s:Latest News|b:The new version is now available.|F:fileadmin/summary.pdf|F:fileadmin/detail.pdf" AS _sendmail

	# Two files and a webpage (converted to PDF) are attached.
	10.sql = SELECT "t:john.doe@example.com|f:company@example.com|s:Latest News|b:The new version is now available.|F:fileadmin/summary.pdf|F:fileadmin/detail.pdf|p:?id=export&r=123|d:person.pdf" AS _sendmail

	# Two webpages (converted to PDF) are attached.
	10.sql = SELECT "t:john.doe@example.com|f:company@example.com|s:Latest News|b:The new version is now available.|p:?id=export&r=123|d:person123.pdf|p:?id=export&r=234|d:person234.pdf" AS _sendmail

	# One file and two webpages (converted to PDF) are *concatenated* to one PDF and attached.
	10.sql = SELECT "t:john.doe@example.com|f:company@example.com|s:Latest News|b:The new version is now available.|C|F:fileadmin/summary.pdf|p:?id=export&r=123|p:?id=export&r=234|d:complete.pdf" AS _sendmail

	# One T3 webpage, protected by a SIP, are attached.
	10.sql = SELECT "t:john.doe@example.com|f:company@example.com|s:Latest News|b:The new version is now available.|p:?id=export&r=123&_sip=1|d:person123.pdf" AS _sendmail

.. _column_img:

Column: _img
^^^^^^^^^^^^

Renders images. Allows to define an alternative text and a title attribute for the image. Alternative text and title text are optional.

*   If no alternative text is defined, an empty alt attribute is rendered in the img tag (since this attribute is mandatory in HTML).
*   If no title text is defined, the title attribute will not be rendered at all.

**Syntax**

::


    10.sql = SELECT "<path to image>|[<alt text>]|[<title text>]" AS _img

..



+-------------+-------------------------------------------------------------------------------------------+---------------------------+
|**Parameter**|**Description**                                                                            |**Default value/behaviour**|
+=============+===========================================================================================+===========================+
|<pathtoimage>|The path to the image file.                                                                |none                       |
+-------------+-------------------------------------------------------------------------------------------+---------------------------+
|<alttext>    |Alternative text. Will be displayed if image can't be loaded (alt attribute of img tag).   |empty string               |
+-------------+-------------------------------------------------------------------------------------------+---------------------------+
|<titletext>  |Text that will be set as image title in the title attribute of the img tag.                |no title attribute rendered|
+-------------+-------------------------------------------------------------------------------------------+---------------------------+


**Minimal Example**

::


    10.sql = SELECT "fileadmin/img/img.jpg" AS _img

..



**Advanced Examples**

::


    10.sql = SELECT "fileadmin/img/img.jpg|Aternative Text" AS _img            # alt="Alternative Text, no title
    20.sql = SELECT "fileadmin/img/img.jpg|Aternative Text|" AS _img           # alt="Alternative Text, no title
    30.sql = SELECT "fileadmin/img/img.jpg|Aternative Text|Title Text" AS _img # alt="Alternative Text, title="Title Text"
    40.sql = SELECT "fileadmin/img/img.jpg|Alternative Text" AS _img           # alt="Alternative Text", no title
    50.sql = SELECT "fileadmin/img/img.jpg" AS _img                            # empty alt, no title
    60.sql = SELECT "fileadmin/img/img.jpg|" AS _img                           # empty alt, no title
    70.sql = SELECT "fileadmin/img/img.jpg||Title Text" AS _img                # empty alt, title="Title Text"
    80.sql = SELECT "fileadmin/img/img.jpg||" AS _img                          # empty alt, no title

..

.. _column_exec:

Column: _exec
^^^^^^^^^^^^^

Runs batch files or executables on the webserver. In case of an error, returncode and errormessage will be returned.

**Syntax**

::


    <command>

..



+-------------+--------------------------------------------------+-----------------+
|**Parameter**|**Description**                                   |**Default value**|
+=============+==================================================+=================+
|<command>    |The command that should be executed on the server.|none             |
+-------------+--------------------------------------------------+-----------------+


**Minimal Examples**

::


    10.sql = SELECT "ls -s" AS _exec
    20.sql = SELECT "./batchfile.sh" AS _exec

..

.. _column_pdf:

Column: _pdf | _file | _zip
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Detailed explanation: download_

Most of the other Link-Class attributes can be used to customize the link. ::

    10.sql = SELECT "[options]" AS _pdf, "[options]" AS _file, "[options]" AS _zip

    with: [options] = [d:<exportFilename][|p:<params>][|U:<params>][|u:<url>][|F:file][|t:<text>][|a:<message>][|o:<tooltip>][|c:<class>][|r:<render mode>]


* Parameter are position independent.
* *<params>*: see `download-parameter-files`_
* For column `_pdf` and `_zip`, the element sources `p:...`, `U:...`, `u:...`, `F:...` might repeated multiple times.
* Example: ::

		10.sql = SELECT "F:fileadmin/test.pdf" as _pdf,  "F:fileadmin/test.pdf" as _file,  "F:fileadmin/test.pdf" as _zip
		10.sql = SELECT "p:id=export&r=1" as _pdf,  "p:id=export&r=1" as _file,  "p:id=export&r=1" as _zip

		10.sql = SELECT "t:Download PDF|F:fileadmin/test.pdf" as _pdf,  "t:Download PDF|F:fileadmin/test.pdf" as _file,  "t:Download ZIP|F:fileadmin/test.pdf" as _zip
		10.sql = SELECT "t:Download PDF|p:id=export&r=1" as _pdf,  "t:Download PDF|p:id=export&r=1" as _file,  "t:Download ZIP|p:id=export&r=1" as _zip

		10.sql = SELECT "d:complete.pdf|t:Download PDF|F:fileadmin/test1.pdf|F:fileadmin/test2.pdf" as _pdf, "d:complete.zip|t:Download ZIP|F:fileadmin/test1.pdf|F:fileadmin/test2.pdf" as _zip

		10.sql = SELECT "d:complete.pdf|t:Download PDF|F:fileadmin/test.pdf|p:id=export&r=1|u:www.example.com" AS _pdf

.. _column_ppdf:

Column: _Pdf | _File | _Zip
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Detailed explanation: download_

A limited set of attributes is supported: ::

    10.sql = SELECT "[options]" AS _Pdf, "[options]" AS _File, "[options]" AS _Zip

    with: [options] = [<exportFilename>]|[<text>]|[<1: urlparam|file>]|[<2: urlparam|file>]| ... |[<n: urlparam|file>]

* Parameter are position dependent and therefore without a qualifier!
* `<urlparam>` or `<file>` might be given.
* If there is more than one `<urlparam|file>`, only `_Pdf` or `_Zip` is possible.
* Example: ::

		SELECT "||fileadmin/test.pdf" AS _File, "||fileadmin/test.pdf" AS _Pdf, "||fileadmin/test.pdf" AS _Zip
		SELECT "output.pdf|Download PDF|fileadmin/test.pdf" AS _File, "output.pdf|Download PDF|fileadmin/test.pdf" AS _Pdf, "output.zip|Download ZIP|fileadmin/test.pdf" AS _Zip

		SELECT "complete.pdf|Download PDF|fileadmin/test1.pdf|fileadmin/test2.pdf|id=export&r=1" AS _Pdf


.. _column-save-pdf:

Column: _savePdf
^^^^^^^^^^^^^^^^

Generated PDFs can be stored directly on the server with this functionality. The link query consists of the following parameters:

* One or more element sources (such as `F:`, `U:`, `p:`, see download-parameter-files_), including possible wkhtmltopdf parameters
* The export filename and path as `d:` - for security reasons, this path has to start with *fileadmin/*

Tips:

* Please note that this option does not render anything in the front end, but is executed each time it is parsed.
  You may want to add a check to prevent multiple execution.
* It is not advised to generate the filename with user input for security reasons.
* If the target file already exists it will be overwriten. To save individual files, choose a new filename,
  for example by adding a timestamp.

Examples: ::

	SELECT "d:fileadmin/result.pdf|F:fileadmin/_temp_/test.pdf" AS _savePdf
	SELECT "d:fileadmin/result.pdf|F:fileadmin/_temp_/test.pdf|U:id=test&--orientation=landscape" AS _savePdf


.. _column-thumbnail:

Column: _thumbnail
^^^^^^^^^^^^^^^^^^

A thumbnail of the file `T:<pathFilename>` will be rendered and saved with the given pixel size as specified via
`W:<dimension>`. The file is only rendered once and subsequent access is delivered via a local cache. The will be
rendered again, if the source file is newer than the thumbnail or if the thumbnail dimension changes.

The thumbnail pathFilename is a MD5 hash of the pathFilename plus the dimension information.

From multi page files like PDFs, the first page is used as the thumbnail.
All file formats, which GraphicsMagick 'convert' (http://www.graphicsmagick.org/formats.html) supports, can be
used. Office file formats are not supported. Due to speed and quality reasons, SVG files will be converted by inkscape.
If a file format is not known, QFQ tries to show a corresponding file type image provided by Typo3 - such an image is not
scaled.

In configuration_ the exact location of `convert` and `inkscape` can be configured (optional) as well as the directory
names for the cached thumbnails.

+-------+--------------------------------+----------------------------------------------------------------------------+
| Token | Example                        | Comment                                                                    |
+=======+================================+============================================================================+
| T     | T:fileadmin/file3.pdf          | File render a thumbnail                                                    |
+-------+--------------------------------+----------------------------------------------------------------------------+
| W     | W:200x, W:x100, W:200x100      | Dimension of the thumbnail: '<width>x<height>. Both                        |
|       |                                | parameter are otional. If non is given the default is W:150x               |
+-------+--------------------------------+----------------------------------------------------------------------------+
| s     | s:1, s:0                       | Optional. Default: `s:1`. If SIP is enabled, the rendered URL              |
|       |                                | is a link via `api/download.php?..`. Else a direct pathFilename.           |
+-------+--------------------------------+----------------------------------------------------------------------------+
| r     | r:7                            | Render Mode. Default 'r:0'. With 'r:7' only the url will be delivered.     |
+-------+--------------------------------+----------------------------------------------------------------------------+

The render mode '7' is useful, if the URL of the thumbnail have to be used in another way than the provided html-'<img>'
tag. Something like `<body style="background-image:url(bgimage.jpg)">` could be solved with
`SELECT "<body style="background-image:url(", 'T:fileadmin/file3.pdf' AS _thumbnail, ')">'`

Example: ::

	# SIP protected, IMG tag, thumbnail width 150px
	10.sql = SELECT 'T:fileadmin/file3.pdf' AS _thumbnail

	# SIP protected, IMG tag, thumbnail width 50px
	20.sql = SELECT 'T:fileadmin/file3.pdf|W:50x' AS _thumbnail

	# No SIP protection, IMG tag, thumbnail width 150px
	30.sql = SELECT 'T:fileadmin/file3.pdf|s:0' AS _thumbnail

	# SIP protected, only the URL to the image, thumbnail width 150px
	40.sql = SELECT 'T:fileadmin/file3.pdf|s:1|r:7' AS _thumbnail


Dimension
'''''''''

GraphicsMagick support various settings to force the thumbnail size. See http://www.graphicsmagick.org/GraphicsMagick.html#details-geometry.

Cleaning
''''''''

By default, the thumbnail directories are never cleaned. It's a good idea to install a cronjob which purges all files
older than 1 year: ::

	find /path/to/files -type f -mtime +365 -delete

Render
''''''

`Public` thumbnails are rendered at the time when the T3 QFQ record is executed. `Secure` thumbnails are rendered when the
'download.php?s=...' is called. The difference is, that the 'public' thumbnails blocks the page load until all thumbnails
are rendered, instead the `secure` thumbnails are loaded asynchonous via the browser - the main page is already delivered to
browser, all thumbnails appearing after a time.

A way to *pre render* thumbnails, is a periodically called (hidden) T3 page, which iterates over all new uploaded files and
triggers the rendering via column `_thumbnail`.

Thumbnail: secure
'''''''''''''''''

Mode 'secure' is activated via enabling SIP (`s:1`, default). The thumbnail is saved under the path `thumbnailDirSecure`
as configured in configuration_.

The secure path needs to be protected against direct file access by the webmaster / webserver configuration too.

QFQ returns a HTML 'img'-tag: ::

	<img src="api/download.php?s=badcaffee1234">

Thumbnail: public
'''''''''''''''''

Mode 'public' has to be explicit activated by specifying `s:0`. The thumbnail is saved under the path `thumbnailDirPublic`
as configured in configuration_.

QFQ returns a HTML 'img'-tag: ::

  <img src="{{thumbnailDirPublic:Y}}/<md5 hash>.png">

.. _column-monitor:

Column: _monitor
^^^^^^^^^^^^^^^^

Detailed explanation: monitor_

**Syntax** ::

    10.sql = SELECT 'file:<filename>|tail:<number of last lines>|append:<0 or 1>|interval:<time in ms>|htmlId:<id>' AS _monitor

+-------------+-------------------------------------------------------------------------------------------+---------------------------+
|**Parameter**|**Description**                                                                            |**Default value/behaviour**|
+=============+===========================================================================================+===========================+
|<filename>   |The path to the file. Relative to T3 installation directory or absolute.                   |none                       |
+-------------+-------------------------------------------------------------------------------------------+---------------------------+
|<tail>       |Number of last lines to show                                                               |30                         |
+-------------+-------------------------------------------------------------------------------------------+---------------------------+
|<append>     |0: Retrieved content replaces current. 1: Retrieved content will be added to current.      |0                          |
+-------------+-------------------------------------------------------------------------------------------+---------------------------+
|<htmlId>     |Reference to HTML element to whose content replaced by the retrieve one.                   |monitor-1                  |
+-------------+-------------------------------------------------------------------------------------------+---------------------------+

.. _copyToClipboard:

Copy to clipboard
^^^^^^^^^^^^^^^^^

+-------------------+--------------------------------+----------------------------------------------------------------------------+
| Token             | Example                        | Comment                                                                    |
+===================+================================+============================================================================+
| y[:<content>]     | y,  y:some content             | Initiates 'copy to clipboard' mode. Source might given text or page or url |
+-------------------+--------------------------------+----------------------------------------------------------------------------+
| F:<pathFilename>  | F:fileadmin/protected/data.R   | pathFilename in DocumentRoot                                               |
+-------------------+--------------------------------+----------------------------------------------------------------------------+

Example: ::

    10.sql = SELECT 'y:hello world (yank)|t:content direct (yank)' AS _yank,
                    'y:hello world (link)|t:content direct (link)' AS _link,
                    CONCAT('F:', p.pathFileName,'|t:File (yank)|o:', p.pathFileName) AS _yank,
                    CONCAT('y|F:', p.pathFileName,'|t:File (link)|o:', p.pathFileName) AS _link
                FROM Person AS p
                  

.. _download:

Download
--------

Download offers:

* Single file - download a single file (any type),
* PDF create - one or concatenate several files (uploaded) and/or web pages (=HTML to PDF) into one PDF output file,
* ZIP archive - filled with several files ('uploaded' or 'HTML to PDF'-converted).
* Excel - created from scratch or fill a template xlsx with database values.

The downloads are SIP protected. Only the current user can use the link to download files.

By using the `_link` column name:

* the option `d:...` initiate creating the download link and optional specifies an export filename,
* the optional `M:...` (Mode) specifies the export type (file, pdf, zip, export),
* setting `s:1` is mandatory for the download function,
* the alttext `a:...` specifies a message in the download popup.

By using `_pdf`,  `_Pdf`, `_file`, `_File`, `_zip`, `_Zip`, `_excel` as column name, the options `d`, `M` and `s`
will be set.

All files will be read by PHP - therefore the directory might be protected against direct web access. This is the
preferred option to offer secure downloads via QFQ.

In case the download needs a persistant URL (no SIP, no user session), a regular
link, pointing directly to a file, have to be used - the download functionality described here is not appropriate for
such a scenario. If necessary, column-save-pdf_ can be used to generate such a file.

.. _download-parameter-files:

Parameter and (element) sources
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* *download*: `d[:<exportFilename>]`

  * *exportFilename* = <filename for save as> - Name, offered in the 'File save as' browser dialog. Default: 'output.<ext>'.

    If there is no `exportFilename` defined, then the original filename is taken (if there is one, else: output...).

    The user typically expects meaningful and distinct file names for different download links.

* *popupMessage*: `a:<text>` - will be displayed in the popup window during download. If the creating/download is fast, the window might disappear quickly.

* *mode*: `M:<mode>`

  * *mode* = <file | pdf | zip | excel>

      * If `M:file`, the mime type is derived dynamically from the specified file. In this mode, only one element source
        is allowed per download link (no concatenation).

      * In case of multiple element sources, only `pdf`, `zip` and `excel` (template mode) is supported.
      * If `M:zip` is used together with `p:...`, `U:...` or `u:..`, those HTML pages will be converted to PDF. Those files
        get generic filenames inside the archive.
      * If not specified, the **default** 'Mode' depends on the number of specified element sources (=file or web page):

        * If only one `file` is specified, the default is `file`.
        * If there is a) a page defined or b) multiple elements, the default is `pdf`.

* *element sources* - for `M:pdf` or `M:zip`, all of the following three element sources might be specified multiple times.
    Any combination and order of the three options are allowed.

  * *file*: `F:<pathFileName>` - relative or absolute pathFileName offered for a) download (single), or to be concatenated
    in a PDF or ZIP.
  * *page*: `p:id=<t3 page>&<key 1>=<value 1>&<key 2>=<value 2>&...&<key n>=<value n>`.

    * By default, the options given to wkhtml will *not* be encoded by a SIP!
    * To encode the parameter via SIP: Add '_sip=1' to the URL GET parameter.

      E.g. `p:id=form&_sip=1&form=Person&r=1`.

      In that way, specific sources for the `download` might be SIP encrypted.

    * Any current HTML cookies will be forwarded to/via `wkhtml`. This includes the current FE Login as well as any
      QFQ session. Also the current User-Agent are faked via the `wkhtml` page request.

    * If there are trouble with accessing FE_GROUP protected content, please check `wkhtmltopdf`_.

  * *url*: `u:<url>` - any URL, pointing to an internal or external destination.

  * *WKHTML Options* for `page`, `urlParam` or `url`:

    * The 'HTML to PDF' will be done via `wkhtmltopdf`.
    * All possible options, suitable for `wkhtmltopdf`, can be submitted in the `p:...`, `u:...` or `U:...` element source.
      Check `wkhtmltopdf.txt <https://wkhtmltopdf.org/usage/wkhtmltopdf.txt>`_ for possible options. Be aware that
      key/value tuple in the  documentation is separated by a space, but to respect the QFQ key/value notation of URLs,
      the key/value tuple in `p:...`, `u:...` or `U:...` has to be separated by '='. Please see last example below.
    * If an option contains an '&' it must be escaped with double '\\'. See example.

	Most of the other Link-Class attributes can be used to customize the link as well.

Example `_link`: ::

	# single `file`. Specifying a popup message window text is not necessary, cause a file directly accessed is fast.
	SELECT "d:file.pdf|s|t:Download|F:fileadmin/pdf/test.pdf" AS _link

	# single `file`, with mode
	SELECT "d:file.pdf|M:pdf|s|t:Download|F:fileadmin/pdf/test.pdf" AS _link

	# three sources: two pages and one file
	SELECT "d:complete.pdf|s|t:Complete PDF|p:id=detail&r=1|p:id=detail2&r=1|F:fileadmin/pdf/test.pdf" AS _link

	# three sources: two pages and one file
	SELECT "d:complete.pdf|s|t:Complete PDF|p:id=detail&r=1|p:id=detail2&r=1|F:fileadmin/pdf/test.pdf" AS _link

	# three sources: two pages and one file, parameter to wkhtml will be SIP encoded
	SELECT "d:complete.pdf|s|t:Complete PDF|p:id=detail&r=1&_sip=1|p:id=detail2&r=1&_sip=1|F:fileadmin/pdf/test.pdf" AS _link

	# three sources: two pages and one file, the second page will be in landscape and pagesize A3
	SELECT "d:complete.pdf|s|t:Complete PDF|p:id=detail&r=1|p:id=detail2&r=1&--orientation=Landscape&--page-size=A3|F:fileadmin/pdf/test.pdf" AS _link

	# One source and a header file. Note: the parameter to the header URL is escaped with double backslash.
	SELECT "d:complete.pdf|s|t:Complete PDF|p:id=detail2&r=1&--orientation=Landscape&--header={{URL:R}}?indexp.php?id=head\\&L=1|F:fileadmin/pdf/test.pdf" AS _link

..

Example `_pdf`, `_zip`: ::

	# File 1: p:id=1&--orientation=Landscape&--page-size=A3
	# File 2: p:id=form
	# File 3: F:fileadmin/file.pdf
	SELECT 't:PDF|a:Creating a new PDF|p:id=1&--orientation=Landscape&--page-size=A3|p:id=form|F:fileadmin/file.pdf' AS _pdf

	# File 1: p:id=1
	# File 2: u:http://www.example.com
	# File 3: F:fileadmin/file.pdf
	SELECT 't:PDF - 3 Files|a:Please be patient|p:id=1|u:http://www.example.com|F:fileadmin/file.pdf' AS _pdf

	# File 1: p:id=1
	# File 2: p:id=form
	# File 3: F:fileadmin/file.pdf
	SELECT CONCAT('t:ZIP - 3 Pages|a:Please be patient|p:id=1|p:id=form|F:', p.pathFileName) AS _zip

..

Use the `--print-media-type` as wkhtml option to access the page with media type 'printer'. Depending on the website
configuration this switches off navigation and background images.



Rendering PDF letters
^^^^^^^^^^^^^^^^^^^^^

`wkhtmltopdf`, with the header and footer options, can be used to render multi page PDF letters (repeating header,
pagination) in combination with dynamic content. Such PDFs might look-alike official letters, together with logo and signature.

Best practice:

#. Create a clean (=no menu, no website layout) letter layout in a separated T3 branch: ::

	page = PAGE
	page.typeNum = 0
	page.includeCSS {
	  10 = typo3conf/ext/qfq/Resources/Public/Css/qfq-letter.css
	}

	// Grant access to any logged in user or specific development IPs
	[usergroup = *] || [IP = 127.0.0.1,192.168.1.* ]
	  page.10 < styles.content.get
	[else]
	  page.10 = TEXT
	  page.10.value = access forbidden
	[global]

#. Create a T3 `body` page (e.g. page alias: 'letterbody') with some content. Example static HTML content: ::

	<div class="letter-receiver">
	  <p>Address</p>
	</div>
	<div class="letter-sender">
	 <p><b>firstName name</b><br>
	  Phone +00 00 000 00 00<br>
	  Fax +00 00 000 00 00<br>
	 </p>
	</div>

	<div class="letter-date">
	  Zurich, 01.12.2017
	</div>

	<div class="letter-body">
	 <h1>Subject</h1>

	 <p>Dear Mrs...</p>
	 <p>Lucas ipsum dolor sit amet organa solo skywalker darth c-3p0 anakin jabba mara greedo skywalker.</p>

	 <div class="letter-no-break">
	 <p>Regards</p>
	 <p>Company</p>
	 <img class="letter-signature" src="">
	 <p>Firstname Name<br>Function</p>
	 </div>
	</div>

#. Create a T3 letter-`header` page (e.g. page alias: 'letterheader') , with only the header information: ::

		<header>
		<img src="fileadmin/logo.png" class="letter-logo">

		<div class="letter-unit">
		  <p class="letter-title">Department</p>
		  <p>
			 Company name<br>
			 Company department<br>
			 Street<br>
			 City
		  </p>
		</div>
		</header>

#. Create a) a link (Report) to the PDF letter or b) attach the PDF (on the fly rendered) to a mail. Both will call the
   `wkhtml` via the `download` mode and forwards the necessary parameter.

Use in `report`: ::

  sql = SELECT CONCAT('d:Letter.pdf|t:',p.firstName, ' ', p.name,
                       '|p:id=letterbody&pId=', p.id, '&_sip=1&--margin-top=50mm&--margin-bottom=20mm&',
                               '--header-html={{BASE_URL_PRINT:Y}}?id=letterheader&',
                               '--footer-right="Seite: [page]/[toPage]"&',
                               '--footer-font-size=8&--footer-spacing=10') AS _pdf
                FROM Person AS p ORDER BY p.id


Sendmail. Parameter: ::

	sendMailAttachment={{SELECT 'd:Letter.pdf|t:', p.firstName, ' ', p.name, '|p:id=letterbody&pId=', p.id, '&_sip=1&--margin-top=50mm&--margin-bottom=20mm&--header-html={{BASE_URL_PRINT:Y}}?id=letterheader&--footer-right="Seite: [page]/[toPage]"&--footer-font-size=8&--footer-spacing=10' FROM Person AS p WHERE p.id={{id:S}} }}

Replace the static content elements from 2. and 3. by QFQ Content elements as needed: ::

  10.sql = SELECT '<div class="letter-receiver"><p>', p.name AS '_+br', p.street AS '_+br', p.city AS '_+br', '</p>'
            FROM Person AS p WHERE p.id={{pId:S}}


Export area
^^^^^^^^^^^

To offer protected pages, e.g. referenced in download links, the regular FE_GROUPs can't be used, cause the download does
not have the current user privileges (it's a separate process, started as the webserver user).

Create a separated export tree in Typo3 Backend, which is IP access restricted. Only localhost or the FE_GROUP 'admin'
is allowed to access: ::

   [IP = {$tmp.restrictedIPRange} ][usergroup = admin]
      page.10 < styles.content.get
   [else]
      page.10 = TEXT
      page.10.value = Please access from localhost or log in as 'admin' user.
   [global]

.. _excel-export:

Excel export
^^^^^^^^^^^^

Hint: For up/downloading of excel files (no modification), check the generic Form
`input-upload`_ element and the report 'download' (`column_pdf`_) function.

This chapter explains how to create Excel files on the fly.

The Excel file is build in the moment when the user request it by clicking on a
download link.

Mode building:

* `New`: The export file will be completely build from scratch.
* `Template`: The export file is based on an earlier uploaded xlsx file (template). The template itself is unchanged.

Injecting data into the Excel file is done in the same way in both modes: a Typo3 page (rendered without any HTML header
or tags) contains one or more Typo3 QFQ records. Those QFQ records will create plain ASCII output.

If the export file has to be customized (colors, pictures, headlines, ...), the `Template` mode is the preferred option.
IT's much easier to do all cusomizations via Excel and creating a template than by coding in QFQ / Excel export notation.

Setup
'''''

* Create a special column name `_excel` (or `_link`) in QFQ/Report. As a source, define a T3 page, which have to deliver
  the dynamic content (also: `excel-export-sample_`). ::

    SELECT CONCAT('d:final.xlsx|M:excel|s:1|t:Excel (new)|p:?id=exceldata') AS _link

* Create a T3 page which delivers the content.

  * Disable all HTML header and wrapping code on that page. It's also a good idea to limit access to such page on localhost,
    your development network and your webserver address. Typoscript setup: ::

        config.disableAllHeaderCode = 1
        tt_content.stdWrap >
        page >
        page = PAGE

        [usergroup = *] || [IP = 127.0.0.1,192.168.1.*,<webserver IP>]
            page.10 < styles.content.get
        [else]
            page.10 = TEXT
            page.10.value = access forbidden
        [global]

  * Use the regular QFQ Report syntax to create some output.
  * The newline at the end of every line needs to be CHAR(10). To make it simpler, the special column name `... AS _XLS`
    (see _XLS, _XLSs, _XLSb, _XLSn) can be used.
  * One option per line.
  * Empty lines will be skipped.
  * Lines starting with '#' will be skipped (comments). Inline comment signs are NOT recognized as comment sign.
  * Separate <keyword> and <value> by '='.

+-------------+----------------------+---------------------------------------------------------------------------------------------------+
| Keyword     | Example              | Description                                                                                       |
+=============+======================+===================================================================================================+
| 'worksheet' | worksheet=main       | Select a worksheet in case the excel file has multiple of them.                                   |
+-------------+----------------------+---------------------------------------------------------------------------------------------------+
| 'mode'      | mode=insert          | Values: insert,overwrite.                                                                         |
+-------------+----------------------+---------------------------------------------------------------------------------------------------+
| 'position'  | position=A1          | Default is 'A1'. Use the excel notation.                                                          |
+-------------+----------------------+---------------------------------------------------------------------------------------------------+
| 'newline'   | newline              | Start a new row. The column will be the one of the last 'position' statement.                     |
+-------------+----------------------+---------------------------------------------------------------------------------------------------+
| 'str', 's'  | s=hello world        | Set the given string on the given position. The current position will be shift one to the right.  |
|             |                      | If the string contains newlines, option'b' (base64) should be used.                               |
+-------------+----------------------+---------------------------------------------------------------------------------------------------+
| 'b'         | b=aGVsbG8gd29ybGQK   | Same as 's', but the given string has to Base64 encoded and will be decoded before export.        |
+-------------+----------------------+---------------------------------------------------------------------------------------------------+
| 'n'         | n=123                | Set number on the given position. The current position will be shift one to the right.            |
+-------------+----------------------+---------------------------------------------------------------------------------------------------+
| 'f'         | f==SUM(A5:C6)        | Set a formula on the given position. The current position will be shift one to the right.         |
+-------------+----------------------+---------------------------------------------------------------------------------------------------+

Create a output like this: ::

    position=D11
    s=Hello
    s=World
    s=First Line
    newline
    s=Second line
    n=123

This fills D11, E11, F11, D12

In Report Syntax: ::

    # With ... AS _XLS (token explicit given)
    10.sql = SELECT 'position=D10' AS _XLS,
                    's=Hello' AS _XLS,
                    's=World' AS _XLS,
                    's=First Line' AS _XLS,
                    'newline' AS _XLS,
                    's=Second line' AS _XLS,
                    'n=123' AS _XLS,

    # With ... AS _XLSs (token generated internally)
    20.sql = SELECT 'position=D20' AS _XLS,
                    'Hello' AS _XLSs,
                    'World' AS _XLSs,
                    'First Line' AS _XLSs,
                    'newline' AS _XLS,
                    'Second line' AS _XLSs,
                    'n=123' AS _XLS,

    # With ... AS _XLSb (token generated internally and content is base64 encoded)
    30.sql = SELECT 'position=D30' AS _XLS,
                     '<some content with special characters like newline/carriage return>' AS _XLSb

.. _excel-export-sample:

Excel export samples: ::

    # From scratch (both are the same, one with '_excel' the other with '_link')
    SELECT CONCAT('d:new.xlsx|t:Excel (new)|p:?id=exceldata') AS _excel
    SELECT CONCAT('d:new.xlsx|t:Excel (new)|p:?id=exceldata|M:excel|s:1') AS _link

    # Template
    SELECT CONCAT('d:final.xlsx|t:Excel (template)|F:fileadmin/template.xlsx|p:?id=exceldata') AS _excel

    # With parameter (via SIP) - get the Parameter on page 'exceldata' with '{{arg1:S}}' and '{{arg2:S}}'
    SELECT CONCAT('d:final.xlsx|t:Excel (parameter)|p:?id=exceldata&_sip=1&arg1=hello&arg2=world') AS _excel


.. _drag_and_drop:

Drag and drop
-------------

Order elements
^^^^^^^^^^^^^^

Ordering of elements via `HTML5 drag and drop` is supported via QFQ. Any element to order
should be represented by a database record with an order column. If the elements are unordered, they will be ordered after
the first 'drag and drop' move of an element.

Functionality divides into:

* Display: the records will be displayed via QFQ/report.
* Order records: updates of the order column are managed by a specific definition form. The form is not a regular form
  (e.g. there are no FormElements), instead it's only a container to held the SQL update query as well as providing
  access control via SIP. The form is automatically called via AJAX.

Part 1: Display list
''''''''''''''''''''

Display the list of elements via a regular QFQ content record. All 'drag and drop' elements together have to be nested by a HTML
element. Such HTML element:

* With `class="qfq-dnd-sort"`.
* With a form name: `{{'form=<form name>' AS _data-dnd-api}}` (will be replaced by QFQ)
* Only *direct* children of such element can be dragged.
* Every children needs a unique identifier `data-dnd-id="<unique>"`. Typically this is the corresponding record id.
* The record needs a dedicated order column, which will be updated through API calls in time.

A `<div>` example HTML output (HTML send to the browser): ::

    <div class="qfq-dnd-sort" data-dnd-api="typo3conf/ext/qfq/qfq/api/dragAndDrop.php?s=badcaffee1234">
        <div class="anyClass" id="<uniq1>" data-dnd-id="55">
            Numbero Uno
        </div>
        <div class="anyClass" id="<uniq2>" data-dnd-id="18">
            Numbero Deux
        </div>
        <div class="anyClass" id="<uniq3>" data-dnd-id="27">
            Numbero Tre
        </div>
    </div>


A typical QFQ report which generates those `<div>` HTML: ::

    10 {
      sql = SELECT '<div id="anytag-', n.id,'" data-dnd-id="', n.id,'">' , n.note, '</div>'
                   FROM Note AS n
                   WHERE grId=28
                   ORDER BY n.ord

      head = <div class="qfq-dnd-sort" {{'form=dndSortNote&grId=28' AS _data-dnd-api}}">
      tail = </div>
    }


A `<table>` based setup is also possible. Note the attribute  `data-columns="3"` - this generates a dropzone
which is the same column width as the outer table. ::

    <table>
        <tbody class="qfq-dnd-sort" data-dnd-api="typo3conf/ext/qfq/qfq/api/dragAndDrop.php?s=badcaffee1234" data-columns="3">
            <tr> class="anyClass" id="<uniq1>" data-dnd-id="55">
                <td>Numbero Uno</td><td>Numbero Uno.2</td><td>Numbero Uno.3</td>
            </tr>
            <tr class="anyClass" id="<uniq2>" data-dnd-id="18">
                <td>Numbero Deux</td><td>Numbero Deux.2</td><td>Numbero Deux.3</td>
            </tr>
            <tr class="anyClass" id="<uniq3>" data-dnd-id="27">
                <td>Numbero Tre</td><td>Numbero Tre.2</td><td>Numbero Tre.3</td>
            </tr>
        </tbody>
    </table>

A typical QFQ report which generates this HTML: ::

    10 {
      sql = SELECT '<tr id="anytag-', n.id,'" data-dnd-id="', n.id,'" data-columns="3">' , n.id AS '_+td', n.note AS '_+td', n.ord AS '_+td', '</tr>'
                   FROM Note AS n
                   WHERE grId=28
                   ORDER BY n.ord

      head = <table><tbody class="qfq-dnd-sort" {{'form=dndSortNote&grId=28' AS _data-dnd-api}} data-columns="3">
      tail = </tbody><table>
    }

Show / update order value in the browser
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

The 'drag and drop' action does not trigger a reload of the page. In case the order number is shown and the user does
a 'drag and drop', the order number shows the old. To update the dragable elements with the latest order number, a
predefined html id has to be assigned them. After an update, all changed order number (referenced by the html id) will
be updated via AJAX.

The html id per element is defined by `qfq-dnd-ord-id-<id>` where `<id>` is the record id. Same example as above, but
with an updated `n.ord` column: ::

    10 {
      sql = SELECT '<tr id="anytag-', n.id,'" data-dnd-id="', n.id,'" data-columns="3">' , n.id AS '_+td', n.note AS '_+td',
                   '<td id="qfq-dnd-ord-id-', n.id, '">', n.ord, '</td></tr>'
                   FROM Note AS n
                   WHERE grId=28
                   ORDER BY n.ord

      head = <table><tbody class="qfq-dnd-sort" {{'form=dndSortNote&grId=28' AS _data-dnd-api}} data-columns="3">
      tail = </tbody><table>
    }

Part 2: Order records
'''''''''''''''''''''

A dedicated `Form`, without any `FormElements`, is used to define the reorder logic (database update definition).

Fields:

* Name: <custom form name> - used in Part 1 in the  `_data-dnd-api` variable.
* Table: <table with the element records> - used to the update the records specified by `dragAndDropOrderSql`.

* Parameter:

+-------------------------------------+--------------------------------------------------------------------------------+
| Attribute                           | Description                                                                    |
+=====================================+================================================================================+
| orderInterval = <number>            | Optional. By default '10'. Might be any number > 0.                            |
+-------------------------------------+--------------------------------------------------------------------------------+
| orderColumn = <column name>         | Optional. By default 'ord'.                                                    |
+-------------------------------------+--------------------------------------------------------------------------------+
| dragAndDropOrderSql =                                 | Query to selects the *same* records as the report in the     |
| {{!SELECT n.id AS id, n.ord AS ord FROM Note AS n     | same *order!* Inconsistencies results in order differences.  |
| ORDER BY n.ord}}                                      | The columns `id` and `ord` are *mandatory.*                  |
+-------------------------------------------------------+--------------------------------------------------------------+

The form related to the example of part 1 ('div' or 'table'): ::

  Form.name: dndSortNote
  Form.table: Note
  Form.parameter: orderInterval = 1
  Form.parameter: orderColumn = ord
  Form.parameter: dragAndDropOrderSql = {{!SELECT n.id AS id, n.ord AS ord FROM Note AS n WHERE n.grId={{grId:S0}} ORDER BY n.ord}}

Re-Order:

QFQ iterates over the result set of `dragAndDropOrderSql`. The value of column `id` have to correspond to the dragged HTML
 element (given by `data-dnd-id`). Reordering always start with `orderInterval` and is incremented by `orderInterval` with each
 record of the result set. The client reports a) the id of the dragged HTML element, b) the id of the hovered element and
 c) the dropped position of above or below the hovered element. This information is compared to the result set and
 changes are applied where appropriate.

 Take care that the query of part 1 (display list) does a) select the same records and b) in the same order as the query
 defined in part 2 (order records) via `dragAndDropOrderSql`.

 If you find that the reorder does not work at expected, those two sql queries are not identical.

QFQ CSS Classes
---------------

* `qfq-table-50`, `qfq-table-80`, `qfq-table-100` - set min-width and column width to 'auto'
* Background Color: `qfq-color-grey-1`, `qfq-color-grey-2`  (table, row, cell)
* `qfq-100`: Makes an element 'width: 100%'.
* `qfq-left`: Text align left.

Bootstrap
---------

* Table: `table`
* Table > hover: `table-hover`
* Table > condensed: `table-condensed`

E.g.::

  10.sql = SELECT id, name, firstName, ...
  10.head = <table class='table table-condensed qfq-table-50'>

* `qfq-100`, `qfq-left` - makes e.g. a button full width and aligns the text left. ::

    10.sql = SELECT "p:home&r=0|t:Home|c:qfq-100 qfq-left" AS _pagev

Tablesorter
-----------

QFQ includes a third-party client-side table sorter: https://mottie.github.io/tablesorter/docs/index.html

To turn any table into a sortable table:

* Ensure that your QFQ installation is importing the appropriate js/css files, see setup-css-js_.
* Add the `class="tablesorter"` to your `<table>` element.
* Make sure your `<table>` has a `<thead>` and `<tbody>`.

In addition to the *tablesorter* class, there are the following additional options:

* Class `tablesorter-filter` enables row filtering.
* Class `tablesorter-pager` adds table paging functionality. A page navigation
  is dynamically injected.
* Class `tablesorter-column-selector` adds a column selector widget.

Customization:

* Add the desired classes or data attributes to your table html, e.g.:

  * `data-sorter="false"` on a '<th>' to disable sorting on that column
  * `class="filter-false"` on a '<th>' to hide the filter field for that column
  * see docs for more options: https://mottie.github.io/tablesorter/docs/index.html

* You can pass in a default configuration object for the main `tablesorter()` function by using the attribute
  `data-tablesorter-config` on the table.
  Use JSON syntax when passing in your own configuration, such as: ::

    data-tablesorter-config='{"theme":"bootstrap","widthFixed":true,"headerTemplate":"{content} {icon}","dateFormat":"ddmmyyyy","widgets":["uitheme","filter","saveSort","columnSelector"],"widgetOptions":{"filter_columnFilters":true,"filter_reset":".reset","filter_cssFilter":"form-control","columnSelector_mediaquery":false} }'

* If the above customization options are not enough, you can output your own HTML for the pager and/or column selector,
  as well as your own `$(document).ready()` function with the desired config. In this case, it is recommended not to
  use the above *tablesorter* classes since the QFQ javascript code could interfere with your javascript code.

Example: ::

    10 {
      sql = SELECT id, CONCAT('form&form=person&r=', id) AS _Pagee, lastName, title FROM person
      head = <table class="table tablesorter tablesorter-filter tablesorter-pager tablesorter-column-selector">
          <thead><tr><th>Id</th><th data-sorter="false" class="filter-false">Edit</th>
          <th>Name</th><th class="filter-select" data-placeholder="Select a title">Title</th>
          </tr></thead><tbody>
      tail = </tbody></table>
      rbeg = <tr>
      rend = </tr>
      fbeg = <td>
      fend = </td>
    }

.. _monitor:

Monitor
-------

Display a (log)file from the server, inside the browser, which updates automatically by a user defined interval. Access
to the file is SIP protected. Any file on the server is possible.

* On a Typo3 page, define a HTML element with a unique html-id. E.g.: ::

    10.head = <pre id="monitor-1">Please wait</pre>

* On the same Typo3 page, define a SQL column '_monitor' with the necessary parameter: ::

    10.sql = SELECT 'file:fileadmin/protected/log/sql.log|tail:50|append:1|refresh:1000|htmlId:monitor-1' AS _monitor


* Short version with all defaults used to display system configured sql.log: ::

    10.sql = SELECT 'file:{{sqlLog:Y}}' AS _monitor, '<pre id="monitor-1" style="white-space: pre-wrap;">Please wait</pre>'

Report Examples
---------------

The following section gives some examples of typical reports.

Basic Queries
^^^^^^^^^^^^^

*   One simple query

::


    10.sql = SELECT "Hello World"

..



Result:

::


    Hello World

..



    Two simple queries

::


    10.sql = SELECT "Hello World"
    20.sql = SELECT "Say hello"

..



    Result:

::


    Hello WorldSay hello

..



    Two simple queries, with break

::


    10.sql = SELECT "Hello World<br>"
    20.sql = SELECT "Say hello"

..



    Result:

::


    Hello World
    Say hello

..



Accessing the database
^^^^^^^^^^^^^^^^^^^^^^

    Real data, one single column

::


    10.sql = SELECT p.firstName FROM exp_person AS p

..



    Result:

::


    BillieElvisLouisDiana

..



    Real data, two columns

::


    10.sql = SELECT p.firstName, p.lastName FROM exp_person AS p

..



    Result:

::


    BillieHolidayElvisPresleyLouisArmstrongDianaRoss

..



The result of the SQL query is an output, row by row and column by column, without adding any formatting information.
See `Formatting Examples`_ for examples of how the output can be formatted.

Formatting Examples
^^^^^^^^^^^^^^^^^^^

Formatting (i.e. wrapping of data with HTML tags etc.) can be achieved in two different ways:

One can add formatting output directly into the SQL by either putting it in a separate column of the output or by using
concat to concatenate data and formatting output in a single column.

One can use 'level' keys to define formatting information that will be put before/after/between all rows/columns of the
actual levels result.

Two columns

::


    # Add the formatting information as a column
    10.sql = SELECT p.firstName, " " , p.lastName, "<br>" FROM exp_person AS p

..



Result:

::


    Billie Holiday
    Elvis Presley
    Louis Armstrong
    Diana Ross

..



One column 'rend'

::


    10.sql = SELECT p.firstName, " " , p.lastName FROM exp_person AS p
    10.rend = <br>

..

Result:

::


    Billie Holiday
    Elvis Presley
    Louis Armstrong
    Diana Ross

..

More HTML

::


    10.sql = SELECT p.name FROM exp_person AS p
    10.head = <ul>
    10.tail = </ul>
    10.rbeg = <li>
    10.rend = </li>

..

Result: ::

    o Billie Holiday
    o Elvis Presley
    o Louis Armstrong
    o Diana Ross

The same as above, but with braces::

  10 {
    sql = SELECT p.name FROM exp_person AS p
    head = <ul>
    tail = </ul>
    rbeg = <li>
    rend = </li>
  }

Two queries: ::

    10.sql = SELECT p.name FROM exp_person AS p
    10.rend = <br>
    20.sql = SELECT a.street FROM exp_address AS a
    20.rend = <br>

Two queries: nested ::

    # outer query
    10.sql = SELECT p.name FROM exp_person AS p
    10.rend = <br>

    # inner query
    10.10.sql = SELECT a.street FROM exp_address AS a
    10.10.rend = <br>

* For every record of '10', all records of 10.10 will be printed.

Two queries: nested with variables ::

    # outer query
    10.sql = SELECT p.id, p.name FROM exp_person AS p
    10.rend = <br>

    # inner query
    10.10.sql = SELECT a.street FROM exp_address AS a WHERE a.pId='{{10.id}}'
    10.10.rend = <br>

* For every record of '10', all assigned records of 10.10 will be printed.

Two queries: nested with hidden variables in a table ::

    10.sql = SELECT p.id AS _pId, p.name FROM exp_person AS p
    10.rend = <br>

    # inner query
    10.10.sql = SELECT a.street FROM exp_address AS a WHERE a.pId='{{10.pId}}'
    10.10.rend = <br>

Same as above, but written in the nested notation ::

  10 {
    sql = SELECT p.id AS _pId, p.name FROM exp_person AS p
    rend = <br>

    10 {
    # inner query
      sql = SELECT a.street FROM exp_address AS a WHERE a.pId='{{10.pId}}'
      rend = <br>
    }
  }

Best practice *recommendation* for using parameter - see `access-column-values`_ ::

  10 {
    sql = SELECT p.id AS _pId, p.name FROM exp_person AS p
    rend = <br>

    10 {
    # inner query
      sql = SELECT a.street FROM exp_address AS a WHERE a.pId='{{pId:R}}'
      rend = <br>
    }
  }



* Columns starting with a '_' won't be printed but can be accessed as regular columns.

Recent List
^^^^^^^^^^^

A nice feature is to show a list with last changed records. The following will show the 10 last modified (Form or
FormElement) forms: ::

	10 {
	  sql = SELECT CONCAT('p:{{pageAlias:T}}&form=form&r=', f.id, '|t:', f.name,'|o:', GREATEST(MAX(fe.modified), f.modified)) AS _page
				  FROM Form AS f
				  LEFT JOIN FormElement AS fe ON fe.formId = f.id
				  GROUP BY f.id
				  ORDER BY GREATEST(MAX(fe.modified), f.modified) DESC
				  LIMIT 10
	  head = <h3>Recent Forms</h3>
	  rsep = ,&ensp;
	}

.. _`store_user_examples`:

STORE_USER examples
^^^^^^^^^^^^^^^^^^^

Keep variables per user session.

Two pages (pass variable)
'''''''''''''''''''''''''

Sometimes it's useful to have variables per user (=browser session). Set a variable on page 'A' and retrieve the value
on page 'B'.

Page 'A' - set the variable: ::

    10.sql = SELECT 'hello' AS '_=greeting'

Page 'B' - get the value: ::

    10.sql = SELECT '{{greeting:UE}}'

If page 'A' has never been opened with the current browser session, nothing is printed (STORE_EMPTY gives an empty string).
If page 'A' is called, page 'B' will print 'hello'.

One page (collect variables)
''''''''''''''''''''''''''''

A page will be called with several SIP variables, but not at all at the same time. To still get all variables at any time: ::

    # Normalize
    10.sql = SELECT '{{order:USE:::sum}}' AS '_=order', '{{step:USE:::5}}' AS _step, '{{direction:USE:::ASC}}' AS _direction

    # Different links
    20.sql = SELECT 'p:{{pageAlias:T}}&order=count|t:Order by count|b|s' AS _link,
                    'p:{{pageAlias:T}}&order=sum|t:Order by sum|b|s' AS _link,
                    'p:{{pageAlias:T}}&step=10|t:Step=10|b|s' AS _link,
                    'p:{{pageAlias:T}}&step=50|t:Step=50|b|s' AS _link,
                    'p:{{pageAlias:T}}&direction=ASC|t:Order by up|b|s' AS _link,
                    'p:{{pageAlias:T}}&direction=DESC|t:Order by down|b|s' AS _link

    30.sql = SELECT * FROM items ORDER BY {{order:U}} {{direction:U}} LIMIT {{step:U}}

Simulate/switch user: feUser
''''''''''''''''''''''''''''

Just set the STORE_USER variable 'feUser'.

All places with `{{feUser:Y}}` has to be replaced by `{{feUser:UY}}`: ::

    # Normalize
    10.sql = SELECT '{{feUser:UT}}' AS '_=feUser'

    # Offer switching feUser
    20.sql = SELECT 'p:{{pageAlias:T}}&feUser=account1|t:Become "account1"|b|s' AS _link,
                    'p:{{pageAlias:T}}&feUser={{feUser:T}}|t:Back to own identity|b|s' AS _link,


Semester switch (remember last choice)
''''''''''''''''''''''''''''''''''''''

A current semester is defined via configuration in STORE_SYSTEM '{{semId:Y}}'. The first column in 10.sql
`'{{semId:SUY}}' AS '_=semId'` saves
the semester to STORE_USER via '_=semId'. The priority 'SUY' takes either the latest choose (STORE_SIP) or reuse the
last used (STORE_USER) or (first time call during browser session) takes the default from config (STORE_SYSTEM): ::

    # Semester switch
    10 {
      sql = SELECT '{{semId:SUY}}' AS '_=semId',
                   CONCAT('p:{{pageAlias:T}}&semId=', sp.id, '|t:', sp.name, '|s|b|G:glyphicon-chevron-left') AS _link,
                           ' <button class="btn disabled ',   IF({{semId:Y0}}=sc.id, 'btn-success', 'btn-default'), '">',sc.name, '</button> ',
                           CONCAT('p:{{pageAlias:T}}&semId=', sn.id, '|t:', sn.name, '|s|b|G:glyphicon-chevron-right|R') AS _link
                       FROM semester AS sc
                       LEFT JOIN semester AS sp ON sp.id=sc.id-1
                       LEFT JOIN semester AS sn ON sc.id+1=sn.id AND sn.show_semester_from<=CURDATE()
                       WHERE sc.id={{semId:SUY}} AND '{{form:SE}}'=''
                       ORDER BY sc.semester_von
      head = <div class="btn-group" style="position: absolute; top: 15px; right: 25px;">
      tail = </div><p></p>
    }

.. _`system`:

System
======

.. _`autocron`:

AutoCron
--------

The `AutoCron` service fires periodically jobs like `open a webpage` (typically a QFQ page which does some database
actions) or `send mail`.

* AutoCron will be triggered via system cron. Minimal time distance therefore is 1 minute. If this is not sufficient,
  any process who starts `.../typo3conf/ext/qfq/qfq/external/autocron.php` via `/usr/bin/php` frequently might be used.

* Custom start time and frequency.
* Per job:

  * If a job still runs and receives the next trigger, the running job will be completed first.
  * If more than one trigger arrives during a run, only one trigger will be processed.
  * If the system misses a run, it will be played as soon as the system is online again.
  * If multiple runs are missed, only one run is fired as soon as the system is online again.

* Running and processed jobs can easily be monitored via *lastRun, lastStatus, nextRun, inProgress*.

Setup
^^^^^

* Setup a system cron entry, typically as the webserver user ('www-data' on debian).
* Necessary privileges:

  * Read for `.../typo3conf/ext/qfq/*`
  * Write, if a logfile should be written (specified per cron job) in the custom specified directory.

Cron task:  ::

  * * * * * /usr/bin/php /var/www/html/typo3conf/ext/qfq/qfq/external/autocron.php

AutoCron Jobs of type 'website' needs the php.ini setting: ::

  allow_url_fopen = On


Create / edit `AutoCron` jobs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a T3 page with a QFQ record (similar to the formeditor). Such page should be access restricted and is only needed
to edit `AutoCron` jobs: ::

    dbIndex={{indexQfq:Y}}
    form={{form:S}}

    10 {
        # Table header.
        sql = SELECT CONCAT('p:{{pageId:T}}&form=cron') AS _pagen, 'id', 'Next run','Frequency','Comment','Last run','In progress', 'Status' FROM (SELECT 1) AS fake WHERE '{{form:SE}}'=''
        head = <table class='table table-hover qfq-table-50'>
        tail = </table>
        rbeg = <thead><tr>
        rend = </tr></thead>
        fbeg = <th>
        fend = </th>

        10 {
        # All Cron Jobs
        sql = SELECT CONCAT('<tr class="',
                            IF(c.lastStatus LIKE 'Error%','danger',''),
                            IF(c.inProgress!=0 AND DATE_ADD(c.inProgress, INTERVAL 10 MINUTE)<NOW(),' warning',''),
                            IF(c.status='enable','',' text-muted'),'" ',

                            IF(c.inProgress!=0 AND DATE_ADD(c.inProgress, INTERVAL 10 MINUTE)<NOW(),'title="inProgress > 10mins"',
                            IF(c.lastStatus LIKE 'Error%','title="Status: Error"','')),
                            '>'),
                        '<td>', CONCAT('p:{{pageId:T}}&form=cron&r=', c.id) AS _pagee, '</td><td>',
                        c.id, '</td><td>',
                        IF(c.nextrun=0,"", DATE_FORMAT(c.nextrun, "%d.%m.%y %H:%i:%s")), '</td><td>',
                        c.frequency, '</td><td>',
                        c.comment, '</td><td>',
                        IF(c.lastrun=0,"", DATE_FORMAT(c.lastrun,"%d.%m.%y %H:%i:%s")), '</td><td>',
                        IF(c.inProgress=0,"", DATE_FORMAT(c.inProgress,"%d.%m.%y %H:%i:%s")), '</td><td>',
                        LEFT(c.laststatus,40) AS '_+pre', '</td><td>',
                        CONCAT('U:form=cron&r=', c.id) AS _paged, '</td></tr>'
                FROM Cron AS c
        ORDER BY c.id
        }
    }


Usage
^^^^^

The system `cron` service will call the `QFQ AutoCron` every minute. `QFQ AutoCron` checks if there is a pending job, by looking
for jobs with `nextRun` <= `NOW()`. All found jobs will be fired - depending on their type, such jobs will send mail(s) or
open a `webpage`. A `webpage` will mostly be a local T3 page with at least one QFQ record on it. Such a QFQ record might
do some manipulation on the database or any other task.

A job with `nextRun`=0 or `inProgress`!=0 won't never be started.

Due to checking `inProgress`, jobs will never run in parallel, even if a job needs more than 1 minute (interval system
cron).

Job: repeating
''''''''''''''

* frequency: '1 MINUTE', '2 DAY', '3 MONTH', ....

After finishing a job, `nextRun` will be increased by `frequency`. If `nextRun` still points in the past, it will be
increased by `frequency` again, until it points to the future.


Job: asynchronous
'''''''''''''''''

* frequency: <empty>

An 'AutoCron' job becomes 'asynchronous' if `frequency` is empty. Then, auto repeating is switched off.

If `nextRun` is > 0 and in the past, the job will be fired. After the job has been done, `nextRun` will be set to 0.

This is useful for jobs which have to be fired from time to time.

To fire such an asynchronous job, just set `nextRun=NOW()` and wait for the next system cron run.

If such a job is running and a new `nextRun=NOW()` is applied, the 'AutoCron' job will be fired again during the next
system cron run.


Type: Mail
''''''''''

At the moment there is a special sendmail notation - this will change in the future.

* `Mail`: ::

  {{!SELECT 'john@doe.com' AS sendMailTo, 'Custom subject' AS sendMailSubject, 'jane@doe.com' AS sendMailFrom, 123 AS sendMailGrId, 456 AS sendMailXId}}

AutoCron will send as many mails as records are selected by the SQL query in field `Mail`. Field `Mail body` provides
the mail text.


Type: Website
'''''''''''''

The page specified in `URL` will be opened.

Optional the output of that page can be logged to a file (take care to have write permissions on that file).

* `Log output to file`=`output.log` - creates a file in the Typo3 host directory.
* `Log output to file`=`/var/log/output.log` - creates a file in `/var/log/` directory.


Also `overwrite` or `append` can be selected for the output file. In case of `append` a file rotation should be setup on
 OS level.

To check for a successful DB connection, it's a good practice to report a custom token on the T3 page / QFQ record like
'DB Connect: ok'. Such a string can be checked via `Pattern to look for on output=/DB Connect: ok/`. The pattern
needs to be written in PHP PCRE syntax. For a simple search string, just surround them with '/'.
If the pattern is found on the page, the job get's 'Ok' - else 'Error - ...'.

Access restriction
;;;;;;;;;;;;;;;;;;

To protect AutoCron pages not to be triggered accidental or by unprivileged access, access to those page tree might be
limited to localhost. Some example Typoscript: ::

	# Access allowed for any logged in user or via 'localhost'
	[usergroup = *] || [IP = 127.0.0.1]
	  page.10 < styles.content.get
	[else]
	  # Error Message
	  page.10 = TEXT
	  page.10.value = <h2>Access denied</h2>Please log in or access this page from an authorized host. Your current IP address:&nbsp;
	  page.20 = TEXT
	  page.20.data = getenv : REMOTE_ADDR
	[global]



AutoCron / website: HTTPS protocol
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

* For `https` the PHP extension `php_openssl` has to be installed.
* All certificates are accepted, even self signed without a correct chain or hostnames, not listed in the certificate.
  This is useful if there is a general 'HTTP >> HTTPS' redirection configured and the website is accessed via `https://localhost/...`

.. _help:

General Tips
============

* Does the error happens on every *page* or only on specific one?
* Does the error happens on every *form* or only on specific one?

Tips:

* On general errors:

	* Always check the Javascript console of your browser, see `javascriptProblem`_.
	* Always check the Webserver logfiles.

QFQ specific
------------

A variable {{<var>}} is empty
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The sanitize rule is violeted and therefore the value has been removed. Set {{<var>:<store>:all}} as a test.

Page is white: no HTML code at all
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This should not happen.

The PHP process stopped at all. Check the Apache error logfile, look for a stacktrace to find the latest function. Send
a bug report.

Problem with query or variables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Specify the required sanitize class. Remember: for STORE_FORM and STORE_CLIENT the default is sanitize class is `digit`.
This means if the variable content is a string, this violates the sanitize class and the variable will not be replaced.

Tip on Form: put the problematic variable or SQL statement in the 'title' or note 'field' of a `FormElement`. This should show
the content. For SQL statements, remove the outer token (e.g. only one curly brace) to avoid triggering SQL: ::

  FE.title: Person { SELECT ... WHERE id={{buggyVar:alnumx}} }

Tip on Report: In case the query did not contain any double ticks, just wrap all but 'SELECT' in double ticks: ::

 Buggy query:  10.sql = SELECT id, ... FROM myTable WHERE status={{myVar}} ORDER BY status
 Debug query:  10.sql = SELECT "id, ... FROM myTable WHERE status={{myVar}} ORDER BY status"



Error read file config.qfq.php: syntax error on line xx
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Check the given line number. If it's a SQL statement, enclose it in single or double ticks.

Logging
-------

General webserver error log
^^^^^^^^^^^^^^^^^^^^^^^^^^^

For apache: `/var/log/apache2/error_log`

Especially if you got a blank page (no rendering at all), this is typically an uncaught PHP error. Check the error message
and report the bug (http://qfq.io > Contact).

Call to undefined function qfq\\mb_internal_encoding()
''''''''''''''''''''''''''''''''''''''''''''''''''''''

Check that all required php modules are installed. See `preparation`_.


Error Messages
--------------

Internal Server Error
^^^^^^^^^^^^^^^^^^^^^

The browser shows a red popup with 'Internal Server Error'. The message is generated in the browser. Happens e.g. an AJAX
request response of QFQ (=Server) is broken. This might happen e.g. if PHP can't start successfully or PHP fails to run
due to  a missing php module or broken configuration.

Oops, an error occurred! Code: 20180612205917761fc593
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You see this message on all places where a QFQ content record should produce some output. Typically the extension fails
to load. If the error message disappears when the QFQ extension is disabled (instead a message `qfq_qfq can't be rendered`
is shown), than QFQ is the problem.

Search the given code in `typo3temp/logs/*`, in this example 20180612205917761fc593. You'll should find a stacktrace with
a more detailed message.

The error might occur if there are problematic characters in config.qfq.php, like single or double ticks inside strings,
wich are not enclosed (correctly).

.. _`sendEmailProblem`:

sendEmail: Error => TLS setup failed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Switch off the TLS encryption. In `configuration`_ specify for *config.sendEMailOptions*: ::

   -o tls=no

.. _`javascriptProblem`:

Javascript problem
------------------

Open the 'Webdeveloper Tools' (FF: F12, Chrome/Opera: Right mouse click > Inspect Element) in your browser, switch to
'console' and reload the page. Inspect the messages.


