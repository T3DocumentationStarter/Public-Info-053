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


.. include:: ../Includes.txt

.. _general:

General
=======

* Project homepage: https://git.math.uzh.ch/typo3/qfq
* Latest relases: https://w3.math.uzh.ch/qfq/


.. _installation:

Installation
============

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

For the `download`_ function, the program `pdftk` is necessary to concatenate PDF files.

Preparation for Ubuntu 14.04::

	sudo apt-get install php5-mysqlnd php5-intl pdftk
	sudo php5enmod mysqlnd
	sudo service apache2 restart

Preparation steps for Ubuntu 16.04::

	sudo apt install php7.0-intl
	sudo apt install pdftk libxrender1        # for PDF and 'HTML to PDF' (wkhtmltopdf)

.. _wkhtmltopdf:

wkhtmltopdf
^^^^^^^^^^^


`wkhtmltopdf`  `<http://wkhtmltopdf.org/>`_ will be used by QFQ to offer 'website print' and 'HTML to PDF' conversion.
The converter is not included in QFQ and has to be manually installed.

* The Ubuntu package `wkhtmltopdf` needs a running Xserver - this does not work on a headless webserver. Best is to
   install the QT version from the named website above.

In `config-qfq-ini`_ specify the:

* installed `wkhtmltopdf` binary,
* the site base URL.

**Important**: To access FE_GROUP protected pages or content, it's necessary to disable the `[FE][lockIP]` check: `wkhtml`
will access the Typo3 page locally and that IP address is different from the client (=user) IP.

Configure via Typo3 Installtool `All configuration > $TYPO3_CONF_VARS['FE']`: ::

   [FE][lockIP] = 0

HTML to PDF conversion
''''''''''''''''''''''

`wkhtmltopdf` converts a website (local or remote) to a (multi)-page PDF file. It's mainly used in `download`_.

Print
'''''

Different browser prints the same page in different variations. To prevent this, QFQ implements a small PHP wrapper
`print.php` with uses `wkhtmltopdf` to convert HTML to PDF.

Provide a `print this page`-link (replace {current pageId})::

	<a href="typo3conf/ext/qfq/qfq/api/print.php?id={current pageId}">Print this page</a>

Any parameter specified after `print.php` will be delivered to `wkhtmltopdf` as part of the URL.

Typoscript code to implement a print link on every page::

	10 = TEXT
	10 {
		wrap = <a href="typo3conf/ext/qfq/qfq/api/print.php?id=...|&type=2"><span class="glyphicon glyphicon-print" aria-hidden="true"></span> Printview</a>
		data = page:uid
	}

Setup
-----

* Install the extension via the Extensionmanager.

  * If you install the extension by manual download/upload and get an error message
    "can't activate extension": rename the downloaded zip file to `qfq.zip` or `qfq_<version>.zip` (e.g. version: 0.9.1).

  * If the Extensionmanager stops after importing: check your memory limit in php.ini.

* Enable the online local-documentation_.
* Copy/rename the file *<Documentroot>/typo3conf/ext/<ext_dir>/config.example.qfq.ini* to
  *<Documentroot>/typo3conf/config.qfq.ini* and configure the necessary values: `config.qfq.ini`_
  The configuration file is outside the extension directory to not loose it during updates.
* Play the SQL File *<ext_dir>/qfq/sql/formEditor.sql* to fill the database with the *FormEditor* records.
* Configure Typoscript to include Bootstrap, jQuery, QFQ javascript and CSS files.

::

	page.meta {
	  X-UA-Compatible = IE=edge
	  X-UA-Compatible.attribute = http-equiv
	  viewport=width=device-width, initial-scale=1
	}

	page.includeCSS {

		file1 = typo3conf/ext/qfq/Resources/Public/Css/bootstrap.min.css
		file2 = typo3conf/ext/qfq/Resources/Public/Css/bootstrap-theme.min.css
		file3 = typo3conf/ext/qfq/Resources/Public/Css/jqx.base.css
		file4 = typo3conf/ext/qfq/Resources/Public/Css/jqx.bootstrap.css
		file5 = typo3conf/ext/qfq/Resources/Public/Css/qfq-bs.css
	}

	page.includeJS {

		file1 = typo3conf/ext/qfq/Resources/Public/JavaScript/jquery.min.js
		file2 = typo3conf/ext/qfq/Resources/Public/JavaScript/bootstrap.min.js
		file3 = typo3conf/ext/qfq/Resources/Public/JavaScript/validator.min.js
		file4 = typo3conf/ext/qfq/Resources/Public/JavaScript/jqx-all.js
		file5 = typo3conf/ext/qfq/Resources/Public/JavaScript/globalize.js
		file6 = typo3conf/ext/qfq/Resources/Public/JavaScript/tinymce.min.js
		file7 = typo3conf/ext/qfq/Resources/Public/JavaScript/EventEmitter.min.js
		file8 = typo3conf/ext/qfq/Resources/Public/JavaScript/qfq.min.js
	}

.. _form-editor:

FormEditor
----------

Setup a *report* to manage all *forms*:

* Create a Typo3 page.
* Set the 'URL Alias' to `form` (default) or the individual defined value in parameter EDIT_FORM_PAGE (config.qfq.ini).
* Insert a content record of type *qfq*.
* In the bodytext insert the following code:

::

	# If there is a form given by SIP: show
	form={{form:S}}

	10 {
		# List of Forms: Do not show this list of forms if there is a form given by SIP.
		# Table header.
		sql = SELECT CONCAT('{{pageId:T}}&form=Form&') as _Pagen, '#', 'Name', 'Title', 'Table', '' FROM (SELECT 1) AS fake WHERE  '{{form:SE}}'=''
		head = <table class="table table-hover qfq-table-50">
		tail = </table>
		rbeg = <thead><tr>
		rend = </tr></thead>
		fbeg = <th>
		fend = </th>

		10 {
			# All forms
			sql = SELECT CONCAT('{{pageId:T}}&form=Form&r=', f.id) as _Pagee, f.id, f.name, f.title, f.tableName, CONCAT('form=Form&r=', f.id) as _Paged FROM Form AS f  ORDER BY f.name
			rbeg = <tr>
			rend = </tr>
			fbeg = <td>
			fend = </td>
		}
	}

.. _config-qfq-ini:

config.qfq.ini
--------------

+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| Keyword                     | Example                                         | Description                                                                |
+=============================+=================================================+============================================================================+
| DB_USER                     | DB_USER=qfqUser                                 | Credentials configured in MySQL                                            |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| DB_PASSWORD                 | DB_PASSWORD=12345678                            | Credentials configured in MySQL                                            |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| DB_SERVER                   | DB_SERVER=localhost                             | Hostname of MySQL Server                                                   |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| DB_NAME                     | DB_NAME=qfq_db                                  | Database name                                                              |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| DB_NAME_TEST                | DB_NAME_TEST=qfq_db_test                        | Used during development of QFQ                                             |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| DB_INIT                     | DB_INIT=set names utf8                          | Global init for using the database.                                        |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| SQL_LOG                     | SQL_LOG=sql.log                                 | Filename to log SQL commands: relative to <ext_dir> or absolute.           |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| SQL_LOG_MODE                | SQL_LOG_MODE=modify                             | *all*: every statement will be logged - this is a lot                      |
|                             |                                                 | *modify*: log only statements who change data                              |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| SHOW_DEBUG_INFO             | SHOW_DEBUG_INFO=auto                            | Possible values: auto|yes|no. For 'auto': If a BE User is logged in,       |
|                             |                                                 | debug information will be shown on the fronend.                            |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| CSS_LINK_CLASS_INTERNA    L | CSS_LINK_CLASS_INTERNAL=internal                | CSS class name of links which points to internal tagets                    |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| CSS_LINK_CLASS_EXTERNAL     | CSS_LINK_CLASS_EXTERNAL=external                | CSS class name of links which points to internal tagets                    |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| CSS_CLASS_QFQ_CONTAINER     |CSS_CLASS_QFQ_CONTAINER=container                | QFQ with own Bootstrap: 'container'.                                       |
|                             |                                                 | QFQ already nested in Bootstrap of mainpage: <empty>                       |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| CSS_CLASS_QFQ_FORM_PILL     |CSS_CLASS_QFQ_FORM_PILL=qfq-color-grey-1         | Wrap around title bar for pills: CSS Class, typically a background color   |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| CSS_CLASS_QFQ_FORM_BODY     |CSS_CLASS_QFQ_FORM_BODY=qfq-color-grey-2         | Wrap around formelements: CSS Class, typically a background color          |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| DATE_FORMAT                 | DATE_FORMAT= yyyy-mm-dd                         | Possible options: yyyy-mm-dd, dd.mm.yyyy                                   |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| FORM_DATA_PATTERN_ERROR     |FORM_DATA_PATTERN_ERROR=please check pa.         | Customizable error message used in validator.js. 'pattern' violation       |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| FORM_DATA_REQUIRED_ERROR    |FORM_DATA_REQUIRED_ERROR=missing value           | Customizable error message used in validator.js. 'required' fields         |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| FORM_DATA_MATCH_ERROR       |FORM_DATA_MATCH_ERROR=type error                 | Customizable error message used in validator.js. 'match' retype mismatch   |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| FORM_DATA_ERROR             |FORM_DATA_ERROR=generic error                    | Customizable error message used in validator.js. 'no specific' given       |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| FORM_BS_COLUMNS             | FORM_BS_COLUMNS=12                              | The whole form will be wrapped in 'col-md-??'. Default is 12 for 100%      |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| FORM_BS_LABEL_COLUMNS       | FORM_BS_LABEL_COLUMNS = 3                       | Default number of BS columns for the 'label'-column                        |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| FORM_BS_INPUT_COLUMNS       | FORM_BS_INPUT_COLUMNS = 6                       | Default number of BS columns for the 'input'-column                        |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| FORM_BS_NOTE_COLUMNS        | FORM_BS_NOTE_COLUMNS = 3                        | Default number of BS columns for the 'note'-column                         |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| FORM_BUTTON_ON_CHANGE_CLASS | FORM_BUTTON_ON_CHANGE_CLASS=alert-info btn-info | Color for save button after modification                                   |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| BASE_URL_PRINT              | BASE_URL_PRINT=http://example.com               | URL where wkhtmltopdf will fetch the HTML (no parameter, those comes later)|
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| WKHTMLTOPDF                 | WKHTMLTOPDF=/usr/bin/wkhtmltopdf                | Binary where to find wkhtmltopdf.                                          |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| EDIT_FORM_PAGE              | EDIT_FORM_PAGE = form                           | T3 Pagealias to edit a form.                                               |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| LDAP_1_RDN                  | LDAP_1_RDN='ou=Admin,ou=example,dc=com'         | Credentials for non-anonymous LDAP access. At the moment only one set of   |
+-----------------------------+-------------------------------------------------+ crendentials is supported.                                                 |
| LDAP_1_PASSWORD             | LDAP_1_PASSWORD=mySecurePassword                |                                                                            |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| ESCAPE_TYPE_DEFAULT         | ESCAPE_TYPE_DEFAULT=s                           | All variables `{{...}}` get this escape class by default                   |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| SECURITY_VARS_HONEYPOT      | SECURITY_VARS_HONEYPOT = email,username,password| If empty: no check. All named variables will rendered as INPUT elements    |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| SECURITY_ATTACK_DELAY       | SECURITY_ATTACK_DELAY = 5                       | If an attack is detected, sleep 'x' seconds and exit PHP process           |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| SECURITY_SHOW_MESSAGE       | SECURITY_SHOW_MESSAGE = true                    | If an attack is detected, show a message                                   |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+
| SECURITY_GET_MAX_LENGTH     | SECURITY_GET_MAX_LENGTH = 32                    | Check that there are no GET vars longer than 'x' chars                     |
+-----------------------------+-------------------------------------------------+----------------------------------------------------------------------------+

Example: *typo3conf/config.qfq.ini*

::

	; To get internal default values, inactivate the option by commenting (= ';') it.
	DB_USER = qfqUser
	DB_SERVER = localhost
	DB_PASSWORD = 12345678
	DB_NAME = qfq_db
	DB_INIT = set names utf8
	SQL_LOG = sql.log
	SHOW_DEBUG_INFO = auto
	CSS_LINK_CLASS_INTERNAL = internal
	CSS_LINK_CLASS_EXT = external
	;CSS_CLASS_QFQ_CONTAINER =
	;CSS_CLASS_QFQ_FORM =
	CSS_CLASS_QFQ_FORM_PILL = qfq-color-grey-1
	CSS_CLASS_QFQ_FORM_BODY = qfq-color-grey-2
	;DATE_FORMAT= yyyy-mm-dd
	;FORM_DATA_PATTERN_ERROR =
	;FORM_DATA_REQUIRED_ERROR =
	;FORM_DATA_MATCH_ERROR =
	;FORM_DATA_ERROR =
	;FORM_BS_COLUMNS = 12
	;FORM_BS_LABEL_COLUMNS = 3
	;FORM_BS_INPUT_COLUMNS = 6
	;FORM_BS_NOTE_COLUMNS = 3
	BASE_URL_PRINT=http://example.com
	WKHTMLTOPDF=/usr/bin/wkhtmltopdf
	;EDIT_FORM_PAGE = form
	;LDAP_1_RDN='ou=Admin,dc=example,dc=com'
	;LDAP_1_PASSWORD=mySecurePassword
	;ESCAPE_TYPE_DEFAULT=s
	;SECURITY_VARS_HONEYPOT=email,username,password
	;SECURITY_ATTACK_DELAY=5
	;SECURITY_SHOW_MESSAGE=true
	;SECURITY_GET_MAX_LENGTH=32

.. _local-documentation:

Local Documentation
-------------------

To render the QFQ reST documentation:

* Take care to have 'unzip' and 'Python setuptools' installed (necessary to run).

Preparation for Ubuntu 16.04::

	sudo apt install unzip python-setuptools python-pip

* Install the extension "Sphinx Python Documentation Generator and Viewer" (sphinx).

  * Execute the update script (symbol 'two arrows as a circle' behind the extension name)
  * Choose 'Sphinx 1.4.4' - click on 'Import'.

* In the Exension Manager open the configuration dialog of the extension 'sphinx'. Activate the 'Sphinx 1.4.4' option and save it.
* On top of the browser window click on the 'question mark' to open the menu, choose 'Sphinx'.
* Show doumentation 'QFQ Extension'

* If you have problems with the rendering, please check: http://mbless.de/blog/2015/01/26/sphinx-doc-installation-steps.html

.. _concept:

Concept
=======

SIPs
----

The following is a technical background information. Not needed to just use QFQ.

The SIPs are uniq timestamps, created/registered on the fly for a specific browser session (=user). Every SIP is
registered on the server (= stored in a PHP Session) and contains one or more key/value pairs. The key/value pairs never leave
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

The title of the QFQ content element will not be rendered. It's only visible in the backend for orientation.

QFQ Keywords (Bodytext)
^^^^^^^^^^^^^^^^^^^^^^^

 +-------------------+---------------------------------------------------------------------------------+
 | Name              | Explanation                                                                     |
 +===================+=================================================================================+
 | form              | Formname defined in ttcontent record bodytext                                   |
 |                   | - Fix. E.g.: **form = person**                                                  |
 |                   | - by SIP: **form = {{form}}**                                                   |
 |                   | - by SQL: **form = {{SELECT c.form FROM conference AS c WHERE c.id={{a:C}} }}** |
 +-------------------+---------------------------------------------------------------------------------+
 | r                 | <record id> The form will load the record with the specified id                 |
 |                   | - Variants: **r = 123**, by SQL: **r = {{SELECT ...}}**                         |
 |                   | - If not specified, the default is '0'                                          |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.db        | Select a DB. Only necessary if a different than the standard DB should be used. |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.fbeg      | Start token for every field (=column)                                           |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.fend      | End token for every field (=column)                                             |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.head      | Start token for whole <level>                                                   |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.tail      | End token for whole <level>                                                     |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.rbeg      | Start token for row.                                                            |
 +-------------------+---------------------------------------------------------------------------------+
 | <level>.rbgd      | Alternating (per row) token                                                     |
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
 | <level>.althead   | If <level>.sql is empty, these token will be rendered                           |
 +-------------------+---------------------------------------------------------------------------------+
 | debugShowBodyText | If ='1' and config.qfq.ini:*showDebugInfo=yes* - shows a tooltip with bodytext  |
 +-------------------+---------------------------------------------------------------------------------+

.. _debug:

Debug
^^^^^

* config.ini: *SHOW_DEBUG_INFO = yes|no|auto*

  * *yes*:

    * Form:

      * For every internal link/button, show tooltips with decoded SIP on mouseover.
      * Shows an 'Edit form'-button (wrench symbol) on a form. The link points to the T3 page with the :ref:`form-editor`.

    * Report: Will be configured per tt-content record.

      *debugShowBodyText = 1*

  * *no*: No debug info.

  * *auto*: Depending if there is a Typo3 BE session, set internally:

    * *SHOW_DEBUG_INFO = yes*  (BE session exist)
    * *SHOW_DEBUG_INFO = no*   (no BE session)


.. _variables:

Variables
---------

Most fields of a form or report specification might contain:

* ''constants'' (=strings), this is the standard use case.
* ''variables'' retrieved from the stores (see below),
* ''SQL statements'' (limited set of),
* or any combination of the above.

* A variable (or SQL) statement is surrounded by curly braces:

  *{{VarName[:<store / prio>[:<sanitize class>[:<escape>]]]}}*

* Example:

  *{{r}}*

  *{{index:FS}}*

  *{{name:FS:alnumx:s}}*

  *{{SELECT name FROM person WHERE id=1234}}*

  *{{SELECT name FROM person WHERE id={{r}} }}*

  *{{SELECT name FROM person WHERE id={{key1:C:alnumx}} }}*

* Leading and trailing spaces inside curly braces are removed.

  * *{{ SELECT "Hello World"   }}* acts as *{{SELECT "Hello World"}}*
  * *{{ varname   }}* acts as *{{varname}}*


* There are several stores, from where to retrieve the value. If a value is not found in one store, the next store is searched,
  until a value is found or there are no more stores available.
* If anywhere along the line an empty string is found, this **is** a value: therefore, the search will stop.
* If no value is found, the value is an <empty string>.

URL Parameter
^^^^^^^^^^^^^

* URL (=GET) Parameter can be used in *forms* and *reports* as variables.
* If a value violates a parameter sanitize class, the value becomes an empty string.


Escape
^^^^^^

Variables used in SQL Statements might cause trouble by using: NUL (ASCII 0), \\n, \\r, \\, ', ", and Control-Z.

To protect the web application the following `escape` types are available:

	* 'm' - `real_escape_string() <http://php.net/manual/en/mysqli.real-escape-string.php>`_ (m = mysql)
	* 'l' - LDAP search filter values will be escaped: `ldap-escape() <http://php.net/manual/en/function.ldap-escape.php>`_ (LDAP_ESCAPE_FILTER).
	* 'L' - LDAP DN values will be escaped. `ldap-escape() <http://php.net/manual/en/function.ldap-escape.php>`_ (LDAP_ESCAPE_DN).
	* 's' - single ticks will be escaped. str_replace() of ' against \\'.
	* 'd' - double ticks will be escaped: str_replace() of " against \\".
	* '-' - no escaping.

* The `escape` type is defined by the fourth parameter of the variable. E.g.: `{{name:FE:alnumx:m}}` (m = mysql).
* It's possible to combine different `escape` types, they will be processed in the order given. E.g. `{{name:FE:alnumx:Ls}}` (L, s).
* Escaping is typically necessary for SQL or LDAP queries.
* Be careful when escaping nested variables. Best is to escape **only** the most outer variable.
* In `config.qfq.ini`_ a global `ESCAPE_TYPE_DEFAULT` can be defined. The configured escape type applies to all substituted
  variables, who do not contain a *specific* escape type.
* Additionally a `defaultEscapeType` can be defined per `Form` (separate field in the Form Editor). This overwrites the
  global definition of `config.qfq.ini`.
* To suppress a default escape type, define the `escape type` = '-' on the specific variable. E.g.: `{{name:FE:alnumx:-}}`.

Sanitize class
^^^^^^^^^^^^^^

* All values in Store *C* (Client=Browser) and store *F* (Form) will be sanitized:
* All `predefined-variable-names`_ have a specific default sanitize class. For these variables, it's not necessary
  to specify a sanitize class.
* All other variables (Store: C, F) get by default the sanitize class defined in the corresponding form. If not defined
  the default class is 'digit'.
* A default sanitize class can be overwritten by individual definition: *{{a:C:all}}*

+------------------+------+-------+-----------------------------------------------------------------------------------------+
| Name             | Form | Query | Pattern                                                                                 |
+==================+======+=======+=========================================================================================+
| **alnumx**       | Form | Query | [A-Za-z][0-9]@-_.,;: /() ÀÈÌÒÙàèìòùÁÉÍÓÚÝáéíóúýÂÊÎÔÛâêîôûÃÑÕãñõÄËÏÖÜŸäëïöüÿç            |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **digit**        | Form | Query | [0-9]                                                                                   |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **numerical**    | Form | Query | [0-9.-+]                                                                                |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **email**        | Form | Query | [a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}                                          |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **min|max**      | Form |       | Compares the value against an lower and upper limit (numeric or string).                |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **min|max date** | Form |       | Compares the value against an lower and upper date or datetime.                         |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **pattern**      | Form |       | Compares the value against a regexp.                                                    |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **allbut**       | Form | Query | All characters allowed, but not [ ]  { } % & \ #. The used regexp: '^[^\[\]{}%&\\#]+$', |
+------------------+------+-------+-----------------------------------------------------------------------------------------+
| **all**          | Form | Query | no sanitizing                                                                           |
+------------------+------+-------+-----------------------------------------------------------------------------------------+


Security
========

All values passed to QFQ will be:

* Checked against max. length and allowed content, on the client and on the server side. On the server side, the check
  happens before any further processing. The 'length' and 'allowed' content is specified per `FormElement`. 'alnumx' is the
  default allowed content for those. Violating the rules will stop the 'save record' process (Form) or result in an empty value (Report).

* Only elements defined in the `Form` definition or requested by `Report` will be processed.

* UTF8 normalized (normalizer::normalize) to unify different ways of composing characters. It's more a database interest
  to work with unified data.

SQL statements are typically fired as `prepared statements` with separated variables.
Further *custom* SQL statements will be defined by the webmaster - those do not use `prepared statements` and might be
affected by SQL injection. To prevent SQL injection, every variable can be escaped with `mysqli::real_escape_string` by
defining the `escape` modifier `m`.

**QFQ notice**:

* Variables passed by the client (=Browser) are untrusted and use the default sanatize class 'digit' (if nothing else is
  specified). If alpha characters are submitted, the content violates `digit` and becomes therefore empty - there is no
  error message. Best is to always use SIP or digits.

Get Parameter
-------------

**QFQ security restriction**:

* GET parameter might contain urlencoded content (%xx). Therefore all GET parameter will be processed by 'urldecode()'.
  As a result a text like '%nn' in GET variables will always be decoded. It's not possible to transfer '%nn' itself.
* GET variables are limited to SECURITY_GET_MAX_LENGTH chars - any violation will stop QFQ.

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
`config.qfq.ini`_ (default:   'username', 'password' and 'email'). On every start of QFQ (form, report, save, ...),
these variables are tested if they are non-empty. If any of the default configured are needed, an explicit variable name
list have to be unconfigured in `config.qfq.ini`_.

**QFQ security restriction**:

* The honeypot variables can't be used in GET or POST as regular HTML input elements - any values of them will terminate QFQ.

Violation
---------

On any violation, QFQ will sleep SECURITY_ATTACK_DELAY seconds (`config.qfq.ini`_) and than exit the running PHP process.
A detected attack leads to a complete white (=empty) page.

If SECURITY_SHOW_MESSAGE = true (`config.qfq.ini`_), at least a message is displayed.

Client Parameter via SIP
------------------------

Links with URL parameters, targeting to the local website, are typically SIP encoded. Instead of transferring the parameter
as part of the URL, only one uniqe GET parameter 's' is appended at the link. The parameter 's' is uniq (equal to a
timestamp) for the user. Assigned variables are stored as a part of the PHP user session on the server.
Two users might have the same value of parameter 's', but the content is completely independet.

Variables needed by Typo3 remains on the link and are not 'sip-encoded'.

Secure direct fileaccess
------------------------

If the application uploads files, mostly it's not necessary and often a security issue, to offer a direct download of
the uploaded files. Best is to create a directory, e.g. `fileadmin/protected` and deny direct access via webbrowser to it.
E.g. for Apache set a htaccess rule: ::

		<Directory /var/www/html/fileadmin/protected>
			Require all denied
		</Directory>

To offer download of those files, use the reserved columnname '_download':`download`_ or variants.


Store
=====

Only variables that are known in a specified store can be substituted.

 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 |Name |Description                                                                             | Content                                                                    |
 +=====+========================================================================================+============================================================================+
 | F   | :ref:`STORE_FORM`: data not saved in database yet.                                     | All native *FormElements*. Recent values from the Browser.                 |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | S   | :ref:`STORE_SIP`: Client parameter 's' will indicate the current SIP, which will be    | sip, r (recordId), form                                                    |
 |     | loaded from the SESSION repo to the SIP-Store.                                         |                                                                            |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | R   | :ref:`STORE_RECORD`: Record - the current record loaded in the form                    | All columns of the current record from the current table                   |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | B   | :ref:`STORE_BEFORE`: Record - the current record loaded in the form before any update  | All columns of the current record from the current table                   |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | P   | Parent record. E.g.: on multi forms the current record of the outer query              | All columns of the MultiSQL Statement from the table for the current row   |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | D   | Default values column : The *table.column* specified *default value*.                  |                                                                            |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | M   | Column type: The *table.column* specified *type*                                       |                                                                            |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | C   | :ref:`STORE_CLIENT`: POST variable, if not found: GET variable                         | Parameter sent from the Client (=Browser).                                 |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | T   | :ref:`STORE_TYPO3`: a) Bodytext (ttcontent record), b) Typo3 internal variables        | See Typo3 tt_content record configuration                                  |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | V   | :ref:`STORE_VARS`: Generic variables                                                   |                                                                            |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | L   | :ref:`STORE_LDAP`: Will be filled on demand during processing of a *FormElement*       | Custom specified list of LDAP attributes                                   |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | 0   | *Zero* - allways value: 0, might be helpful if a variable is empty or undefined and    | Any key                                                                    |
 |     | will be used in an SQL statement.                                                      |                                                                            |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | E   | *Empty* - allways an empty string, might be helpful if a variable is empty or undefined| Any key                                                                    |
 |     | and will be used in an SQL statement                                                   |                                                                            |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
 | Y   | :ref:`STORE_SYSTEM`: a) Database, b) helper vars for logging/debugging:                |                                                                            |
 |     | SYSTEM_SQL_RAW ... SYSTEM_FORM_ELEMENT_COLUMN, c) Any custom fields: CONTACT, HELP, ...|                                                                            |
 +-----+----------------------------------------------------------------------------------------+----------------------------------------------------------------------------+

* Default *<prio>*: *FSRVD* - Form / SIP / Record / Vars / Table definition.
* Hint: Preferable, parameter should be submitted by SIP, not by Client (=URL).

  * Warning: Data submitted via 'Client' can be easily spoofed and altered.
  * Best: Data submitted via SIP never leaves the server, cannot be spoofed or altered by the user.
  * SIPs can _only_ be defined by using *Report*. Inside of *Report* use columns 'Link' (with attribute 's'), 'page?' or 'Page?'.

.. _predefined-variable-names:

Predefined variable names
-------------------------

.. _STORE_FORM:

Store: *FORM* - F
^^^^^^^^^^^^^^^^^

* Sanatized: *yes*
* Represents the values in the form, typically before saving them.
* Used for:

  * *FormElements* who will be rerendered, after a parent *FormElement* has been changed by the user.
  * *FormElement* actions, before saving the form.
  * Values will be sanitized by the class configured in corresponding the *FormElement*. By default, the sanitize class is `alnumx`.

 +---------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                            | Explanation                                                                                                                                |
 +=================================+============================================================================================================================================+
 | <FormElement name>              | Name of native *FormElement*. To get, exactly and only, the specified *FormElement* (for 'pId'): *{{pId:F}}*                               |
 +---------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+

.. _STORE_SIP:

Store: *SIP* - S
^^^^^^^^^^^^^^^^

* Sanatized: *no*
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
 | urlparam                | all non Typo3 paramter in one string                      |
 +-------------------------+-----------------------------------------------------------+
 | <user defined>          | additional user defined link parameter                    |
 +-------------------------+-----------------------------------------------------------+

.. _STORE_RECORD:

Store: *RECORD* - R
^^^^^^^^^^^^^^^^^^^

* Sanatized: *no*
* Current record loaded in Form.
* If r=0, all values are empty.

 +------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                   | Explanation                                                                                                                                      |
 +========================+==================================================================================================================================================+
 | <column name>          | Name of a column of the primary table (as defined in the current form). To get, exactly and only, the specified form *FormElement*: *{{pId:R}}*  |
 +------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------+

.. _STORE_BEFORE:

Store: *BEFORE* - B
^^^^^^^^^^^^^^^^^^^

* Sanatized: *no*
* Current record loaded in Form without any modification.
* If r=0, all values are empty.

This store is handy to compare new and old values of a form.

 +------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                   | Explanation                                                                                                                                      |
 +========================+==================================================================================================================================================+
 | <column name>          | Name of a column of the primary table (as defined in the current form). To get, exactly and only, the specified form *FormElement*: *{{pId:R}}*  |
 +------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------+

.. _STORE_CLIENT:

Store: *CLIENT* - C
^^^^^^^^^^^^^^^^^^^

* Sanatized: *yes*

 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                    | Explanation                                                                                                                              |
 +=========================+==========================================================================================================================================+
 | s                       | =SIP                                                                                                                                     |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | r                       | record id. Typically stored in SIP, rarely specified on the URL                                                                          |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | keySemId                | always current Semester Id                                                                                                               |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | keySemIdUser            | *{{keySemIdUser}}*, may be changed by user                                                                                               |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | HTTP_HOST               | current HTTP HOST                                                                                                                        |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | REMOTE_ADDR             | Client IP address                                                                                                                        |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | '$_SERVER[*]'           | All other variables accessable by *$_SERVER[]*. Only the often used have a pre-defined sanitize class.                                   |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | form                    | Unique name of current form                                                                                                              |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | ANREDE                  | *{{sex}}* == male >> Sehr geehrter Herr, *{{sex}}* == female  Sehr geehrte Frau                                                          |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+
 | EANREDE                 | *{{sex}}* == male >> Dear Mr., *{{sex}}* == female >> Dear Mrs.                                                                          |
 +-------------------------+------------------------------------------------------------------------------------------------------------------------------------------+

.. _STORE_TYPO3:

Store: *TYPO3* (Bodytext) - T
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Sanatized: *no*

 +-------------------------+-------------------------------------------------------------------+----------+
 | Name                    | Explanation                                                       | Note     |
 +=========================+===================================================================+==========+
 | form                    | Formname defined in ttcontent record bodytext                     | see note |
 |                         |                                                                   |          |
 |                         | * Fix. E.g. *form = person*                                       |          |
 |                         | * via SIP. E.g. *form = {{form}}*                                 |          |
 +-------------------------+-------------------------------------------------------------------+----------+
 | pageId                  | Record id of current Typo3 page                                   | see note |
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

* **note**: not available
  * in :ref:`dynamic-update` or
  * by *FormElement* class 'action' with type 'beforeSave', 'afterSave', 'beforeDelete', 'afterDelete'.

.. _STORE_VARS:

Store: *VARS* - V
^^^^^^^^^^^^^^^^^

* Sanatized: *no*

 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                    | Explanation                                                                                                                                |
 +=========================+============================================================================================================================================+
 | random                  | random string with length of 32 chars, alphanum                                                                                            |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | slaveId                 | see *FormElement* `action`                                                                                                                 |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | filename                | Original filename of an uploaded file via an 'upload'-FormElement. Valid only during processing of the current 'upload'-formElement.       |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | fileDestinaton          | Destination (path & filename) for an uploaded file. Defined in an 'upload'-FormElement.parameter. Valid: same as 'filename'.               |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+

.. _STORE_LDAP:

Store: *LDAP* - L
^^^^^^^^^^^^^^^^^

* Sanatized: *yes*
* See also :ref:`LDAP`:

 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
 | Name                    | Explanation                                                                                                                                |
 +=========================+============================================================================================================================================+
 | <custom defined>        | See *ldapAttributes*                                                                                                                       |
 +-------------------------+--------------------------------------------------------------------------------------------------------------------------------------------+



.. _STORE_SYSTEM:

Store: *SYSTEM* - Y
^^^^^^^^^^^^^^^^^^^

* Sanatized: *no*

 +-------------------------+--------------------------------------------------------------------------+
 | Name                    | Explanation                                                              |
 +=========================+==========================================================================+
 | DB_USER                 | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | DB_SERVER               | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | DB_NAME                 | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | DB_INIT                 | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | SQL_LOG                 | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | SQL_LOG_MODE            | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | SHOW_DEBUG_INFO         | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | CSS_LINK_CLASS_INTERNAL | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | CSS_LINK_CLASS_EXTERNAL | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | CSS_CLASS_QFQ_CONTAINER | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | EXT_PATH                | computed during runtime                                                  |
 +-------------------------+--------------------------------------------------------------------------+
 | SITE_PATH               | computed during runtime                                                  |
 +-------------------------+--------------------------------------------------------------------------+
 | DATE_FORMAT             | defined in config.ini                                                    |
 +-------------------------+--------------------------------------------------------------------------+
 | class                   | defined in config.ini (CSS_CLASS_QFQ_FORM) or form definition            |
 +-------------------------+--------------------------------------------------------------------------+
 | classPill               | defined in config.ini (CSS_CLASS_QFQ_FORM_PILL) or form definition       |
 +-------------------------+--------------------------------------------------------------------------+
 | classBody               | defined in config.ini (CSS_CLASS_QFQ_FORM_BODY) or form definition       |
 +-------------------------+--------------------------------------------------------------------------+
 | data-pattern-error      | defined in config.ini or form definition                                 |
 +-------------------------+--------------------------------------------------------------------------+
 | data-require-error      | defined in config.ini or form definition                                 |
 +-------------------------+--------------------------------------------------------------------------+
 | data-match-error        | defined in config.ini or form definition                                 |
 +-------------------------+--------------------------------------------------------------------------+
 | data-error              | defined in config.ini or form definition                                 |
 +-------------------------+--------------------------------------------------------------------------+
 | bsColumns               | defined in config.ini (FORM_BS_COLUMNS) or form definition               |
 +-------------------------+--------------------------------------------------------------------------+
 | bsLabelColumns          | defined in config.ini (FORM_BS_LABEL_COLUMNS) or form definition         |
 +-------------------------+--------------------------------------------------------------------------+
 | bsInputColumns          | defined in config.ini (FORM_BS_INPUT_COLUMNS) or form definition         |
 +-------------------------+--------------------------------------------------------------------------+
 | bsNoteColumns           | defined in config.ini (FORM_BS_NOTE_COLUMNS) or form definition          |
 +-------------------------+--------------------------------------------------------------------------+
 | sqlFinal                | computed during runtime, used for error reporting                        |
 +-------------------------+--------------------------------------------------------------------------+
 | sqlParamArray           | computed during runtime, used for error reporting                        |
 +-------------------------+--------------------------------------------------------------------------+
 | sqlCount                | computed during runtime, used for error reporting                        |
 +-------------------------+--------------------------------------------------------------------------+

SQL Statement
-------------

* The detection of an SQL command is case *insensitive*.
* Leading  whitespace will be skipped.
* The following commands are interpreted as SQL commands:

  * SELECT
  * INSERT, UPDATE, DELETE, REPLACE, TRUNCATE
  * SHOW, DESCRIBE, EXPLAIN, SET

* A SQL Statement might contain parameters, including additional SQL statements. Inner SQL queries will be executed first.
* All variables will be substituted one by one from inner to outer.
* Maximum recursion depth: 5 (a recursion depth of 2 is sometimes used for mailing with templates, 3 and more probably confuses too much and is therefore not practicable, but supported until depth of 5)
* The number of variables inside an input field or a SQL statement is not limited.
* A resultset of a SQL statement will be imploded over all: concat all columns of a row, concat all rows - there is no glue string.

* Example::

  {{SELECT id, name FROM Person}}
  {{SELECT id, name, IF({{feUser}}=0,'Yes','No')  FROM Vorlesung WHERE sem_id={{keySemId:Y}} }}
  {{SELECT id, city FROM Address AS adr WHERE adr.pId={{SELECT id FROM Account AS acc WHERE acc.name={{feUser}} }} }}

* Special case for SELECT input fields. To deliver a result array specify an '!' before the SELECT: ::

   {{!SELECT ...}}

  * This is only possible for the outermost SELECT.

.. _LDAP:

LDAP
====

A form can retrieve data from LDAP server(s) to display or to save them. Configuration options for LDAP will be specified
in the *parameter* field of the *Form* and/or the *FormElement*. Definitions of the *FormElement* will overwrite definitions
of the *Form*. One LDAP Server can be configured per *FormElement*. Multiple *FormElements* might use individual LDAP
Server configurations.

To decide which Parameter should be placed on *Form.parameter* and which on *FormElement.parameter*: If LDAP access is ...

* only necessary in one *FormElement*, most usefull setup is to specify all values in that specific *FormElement*,
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
| ldapSearch                  | (mail=john.doe@example.com)      | Regular LDAP search expresssion                               | x    | x           | FSL      |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| ldapTimeLimit               | 3 (default)                      | Maximum time to wait for an answer of the LDAP Server         | x    | x           | TA, FSL  |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| ldapUseBindCredentials      | ldapUseBindCredentials=1         | Use LDAP_1_* crendentials from config.qfq.ini for ldap_bind() | x    | x           | TA, FSL  |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLdap               | -                                | Enable LDAP as 'Typeahead' data source                        |      | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLdapSearch         | `(|(cn=*?*)(mail=*?*))`          | Regular LDAP search expresssion, returns upto typeAheadLimit  | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadLdapSearchPrefetch | `(mail=?)`                       | Regular LDAP search expresssion, typically return one record  | x    | x           | TA       |
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
| typeAheadPedantic           | typeAheadPedantic                | Activate 'pedantic' mode - only valid keys are allowed        | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| typeAheadMinLength          | 2 (default)                      | Minimum number of characters before starting the search       | x    | x           | TA       |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+
| fillStoreLdap               | -                                | Activate `Fill STORE LDAP` with the first retrieved record    |      | x           | FSL      |
+-----------------------------+----------------------------------+---------------------------------------------------------------+------+-------------+----------+

* *typeAheadLimit*: there might be a hard limit on the server side (e.g. 100) - which can't be extended.
* *ldapUseBindCredentials* is only necessary if `anonymous` access is not possible. RDN and password has to be configured in
  `config.qfq.ini`.

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

All occurences of a '?' in *ldapSearch* will be replaced by the user data typed in via the text-*FormElement*.
The typed data will be escaped to fullfill LDAP search limitations.
Regular *Form* variables might be used on all parameter and will be evaluated during form load (!) - *not* at the time when
the user types something.

Pedantic
^^^^^^^^

In case the typed value (technically this is the value of the *id*, latest in the moment when loosing the focus) have
to be a valid (= exist on the LDAP server), the *typeAheadPedantic* mode can be activated.
If the user typed something and that is not a valid *id*, the client (=browser) will delete the input when loosing the focus.
To identify the exact *id*, an additional search filter is necessary: `ypeAheadLdapSearchPrefetch` - see next topic.

* *Form.parameter* or *FormElement.parameter*:

  * *typeAheadPedantic*

Prefetch
^^^^^^^^

After 'form load' with an existing record, the user epects to see the previous saved data. In case there is an *id* to
*value* translation, the *value* does not exist in the database, instead it has to be fetched again dynamically from the
LDAP server. A precise LDAP query has to be defined to force this:

* *Form.parameter* or *FormElement.parameter*:

  * *typeAheadLdapSearchPrefetch* = `(mail=?)`

This situation also applies in *pedantic* mode to verify the user input after each change.

PerToken
^^^^^^^^

Sometimes a LDAP server only provides attributes like 'sn' and 'givenName', but not 'displayName' or another practial
combination of multiple attributes - than it is difficult to search for 'firstname' *and* (=human AND) 'lastname'.
E.g. 'John Doe', results to search like `(|(sn=*John Doe*)(givenName=*John Doe*))` which will be probably always be empty.
Instead, the user input has to be splitted in token and the search string has to repeated for every token.

* *Form.parameter* or *FormElement.parameter*:

  * *typeAheadLdapSearchPerToken* - no value needed.

This will repeat the search string per token.

E.g.::

   User search string: X Y
   Ldap search string: (|(a=*?*)(b=*?*))

   Result: (& (|(a=*X*)(b=*X*)) (|(a=*Y*)(b=*Y*))

Attention: this option is only usefull in specific environments. Only use it, if it is really needed. The query becomes
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

The FLS happens *before* the main *FormElement* processing starts. Therefore the fetched LDAP data (specified by *ldapAttributes*),
are available via `{{<attributename>:L:allbut:s}}` during the regular *FormElement* processing. Take care to specify
a sanatize class and optional escaping on further processing of those data.

Important: LDAP access might slow down the *Form* processing on load, update or save! The timeout (default: 3 seconds) have
 to be multiplied by the number of accesses. E.g. a broken LDAP connection and 3 *FormELements* with *FSL*
 results to 9 seconds delay on save. Also be prepared not to receive the expected data.

* *FormElement.parameter.fillStoreLdap* - activate the mode *Fill S* - no value is needed, the existence is suffucient.
* *Form.parameter* or *FormElement.parameter*:

  * *ldapServer* = `directory.example.com`
  * *ldapBaseDn* =  `ou=Addressbook,dc=example,dc=com`
  * *typeAheadLdapSearch* = `(|(cn=*?*)(mail=*?*))`
  * *ldapAttributes* = `givenName, sn, telephoneNumber, email`
  * *ldapSearch* = `(mail={{email::l}})`
  * Optional: *ldapUseBindCredentials* = 1

After filling the store, access the content via `{{<attributename>:allbut:L:s}}`.

Form
====

* Forms will be created by using the *QFQ Form Editor* on the Typo3 frontend (HTML form).
* The Formeditor itself consist of two predefined QFQ forms: *form* and *formElement* - these forms are often updated
  during the installation of new QFQ versions.
* Every form consist of a) a *Form* record and b) multiple *FormElement* records.
* A form is assigned to a  *table*. Such a table is called the *primary table* for this form.
* There are three types of forms which can roughly categorized into:

  * *Simple* form: the form acts on one record, stored in one table.

    * The form will create necessary SQL commands for insert, update and delete (only primary record) automatically.

  * *Advanced* form: the form acts on multiple records, stored in more than one table.

    * Fields of the primary table acts like a *simple* form, all other fields have to be specified with *action/afterSave* records.

  * *Multi* form: the form acts simultanously on more than one record. All records use the same *FormElements*.

    * The *FormElements* are defined as a regular *simple* / or *advanced* form, plus a SQL Query, which selects and
      iterates over all records. Those records will be loaded at the same time.

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

* Display a form:

  * Create a QFQ tt_content record on a Typo 3 page.
  * Inside the QFQ record: `form  = <formname>`. E.g.:

    * Static: `form = Person`
    * Dynamic: `form  = {{form:S}}`  (the left `form` is a keyword for QFQ, the right `form` is a free chooseable variable name)

  * With the `Dynamic` option, it's easily possible to use one Typo3 page and display different forms on that specific
    page. This is nice to configure few Typo 3 pages. The disadvantage is that the user might loose the navigation.

.. _form-main:

Definition
----------

+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
| Name                    | Type                                                     | Description                                                                             |
+=========================+==========================================================+=========================================================================================+
|id                       | int, autoincrement                                       | created by by MySQL                                                                     |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|name                     | string                                                   | unique and speaking name of the form. Form will be identified by this name              |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|title                    | string                                                   | Title, shown on/above the form.                                                         |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|noteInternal             | textarea                                                 | Internal notes: special functionality, used variables, ...                              |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|tableName                | string                                                   | Primay table of the form                                                                |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|permitNew                | enum('sip', 'logged_in', 'logged_out', 'always', 'never')| Default: sip                                                                            |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|permitEdit               | enum('sip', 'logged_in', 'logged_out', 'always', 'never')| Default: sip                                                                            |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|render                   | enum('plain','table', 'bootstrap')                       | Default bootstrap                                                                       |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|requiredParameter        | string                                                   | Name of required SIP parameter, seperated by comma. '#' as comment delimiter            |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|showButton               | set('new', 'delete', 'close', 'save')                    | Default 'new,delete,close,save'. Shown buttons in the upper right corner of the form.   |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|multiMode                | enum('none','horizontal','vertical')                     | Default 'none'                                                                          |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|multiSql                 | text                                                     | Optional. SQL Query which selects all records to edit.                                  |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|multiDetailForm          | string                                                   | Optional. Form to open, if a record is selected to edit (double click on record line)   |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|multiDetailFormParameter | string                                                   | Optional. Translated Parameter submitted to detailform (like subrecord parameter)       |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|forwardMode              | string: 'auto | no | page'                               |                                                                                         |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|forwardPage              | string / query                                           | If $forward=="page": page to jump to                                                    |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|bsLabelColumns           | string                                                   | The bootstrap grid system is based on 12 columns. The sum of *bsLabelColumns*,          |
+-------------------------+----------------------------------------------------------+ *bsInputColumns* and *bsNoteColumns* should be 12. These values here are the base values|
|bsInputColumns           | string                                                   | for all *FormElements*. Exceptions per *FormElement* can be specified per *FormElement*.|
+-------------------------+----------------------------------------------------------+ Default: label=3, input=6, note=3. See :ref:`form-layout`.                              |
|bsNoteColumns            | string                                                   |                                                                                         |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|parameter                | text                                                     | Misc additional parameters. See :ref:`form-parameter`.                                  |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|deleted                  | string                                                   | 'yes'|'no'.                                                                             |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|modified                 | timestamp                                                | updated automatically through stored procedure                                          |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+
|created                  | datetime                                                 | set once through QFQ                                                                    |
+-------------------------+----------------------------------------------------------+-----------------------------------------------------------------------------------------+

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


* `sip` is *always* the preferred way. With 'sip' it's not necessary to differ between logged in or not, cause the SIP
   only  exist and is only valid, if it's created via QFQ/report earlier. This means 'creating' the SIP implies
   'access granted'. The grant will be revoked when the QFQ session is destroyed - this happens when a user loggs out or
   the webbrowser is closed.

* `logged_id` / `logged_out`: for forms which might be displayed without a SIP, but maybe on a protected or even unprotected
  page. *The option is probably not often used.*

* `always`: such a form is always allowed to be loaded.

  * `permitNew=always`: Public accessible forms (e.g. for registration) will allow users to fill and save
    the form.

  * `permitEdit=always`: Public accessible forms will allow users to update existing data. This
    is dangerous, cause the URL paramater (recordId) 'r' might be changed by the user (URL manipulating) and therefore
    the user might see and/or change data from other users. *The option is probably not often used.*

* `never`: such a form is not allowed to be loaded.

  * `permitNew=never`: Public accessible forms. It's not possible to create new records.

  * `permitEdit=none`: Public accessible forms. It's not possible to update records.




showButton
^^^^^^^^^^

Display or hide the button `new`, `delete`, `close`, `save`.

* *new*: Creates a new record. If the form needs any special parameter via SIP or Client (=browser), hide this 'new' button - the necessary parameter are not provided.
* *delete*: This either deletes the current record only, or (if defined via action *FormElement* 'before Delete' ) any specified subrecords.
* *close*: Close the current form. If there are changes, a popup opens and ask to save / close / cancel. The last page from the history will be shown.
* *save*: Save the form.

* Default: show all buttons.

.. _form-parameter:

parameter
^^^^^^^^^

* The following parameter are optional and can be configured in the *Form.parameter* field.

+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| Name                        | Type   | Description                                                                                              |
+=============================+========+==========================================================================================================+
| bsColumns                   | int    | Wrap the whole form in '<div class="col-md-??">                                                          |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| maxVisiblePill              | int    | Show pills upto <maxVisiblePill> as button, all further in a drop-down menu. Eg.: maxVisiblePill=3       |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| class                       | string | HTML div with given class, surrounding the whole form. Eg.: class=container-fluid                        |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| classPill                   | string | HTML div with given class, surrounding the `pill` title line.                                            |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| classBody                   | string | HTML div with given class, surrounding all *FormElement*.                                                |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| submitButtonText            | string | Show save button, with the <submitButtonText> at the bottom of the form                                  |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| extraDeleteForm             | string | Name of a form which specifies how to delete the primary record and optional slave records               |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-pattern-error          | string | Pattern violation: Text for error message used for all FormElements of current form                      |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-required-error         | string | Required violation: Text for error message used for all FormElements of current form                     |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-match-error            | string | Match violation: Text for error message used for all FormElements of current form                        |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-error                  | string | If none specific is defined: Text for error message used for all FormElements of current form            |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| buttonOnChangeClass         | string | Color for save button after user modified some content or current form. E.g.: 'btn-info alert-info'      |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapServer                  | string | FQDN Ldap Server. E.g.: directory.example.com                                                            |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapBaseDn                  | string | E.g.: `ou=Addressbook,dc=example,dc=com`                                                                 |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapAttributes              | string | List of attributes to fill STORE_LDAP with. E.g.: cn, email                                              |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapSearch                  | string | E.g.: `(mail={{email::alnumx:l}})`                                                                       |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| ldapTimeLimit               | int    | Maximum time to wait for an answer of the LDAP Server                                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdap               | -      | Enable LDAP as 'Typeahead' data source                                                                   |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdapSearch         | string | Regular LDAP search expresssion. E.g.:  `(|(cn=*?*)(mail=*?*))`                                          |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdapValuePrintf    | string | Value formatting of LDAP result, per entry. E.g.: `'%s / %s / %s', mail, roomnumber, telephonenumber`    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdapIdPrintf       | string | Key formatting of LDAP result, per entry. E.g.: `'%s', mail`                                             |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLdapSearchPerToken | -      | Split search value in token and OR-combine every search with the individual tokens                       |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadLimit              | int    | Maximum number of entries. The limit is applied to the server (LDAP or SQL) and the Client               |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| typeAheadMinLength          | int    | Minimum number of characters which have to typed to start the search.                                    |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| mode                        | string | The value `readonly` will activate a global readonly mode of the form - the user can't change any data.  |
|                             |        | See :ref:`form-mode-readonly`                                                                            |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+
| saveButtonActive            | -      | Make the 'save'-button active on *Form* load (instead of waiting for the first user change)              |
+-----------------------------+--------+----------------------------------------------------------------------------------------------------------+


* Example:

  * maxVisiblePill = 5
  * class = container-fluid
  * classBody = qfq-form-right

.. _comment-space-character:

Comment- and space-character
''''''''''''''''''''''''''''

* Lines will be trimmed - leading and trailing spaces will be removed.
* If a leading and/or trailing space is needed, escape it: '\ hello world \' > ' hello world '.

* Lines starting with a '#' are treated as a comment and will not be parsed. Suche lines are treated as 'empty lines'
* The comment sign can be escaped with '\'


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
* `classPill` is only visible on forms with container elemants of type 'Pill'.

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

.. _form-mode-readonly:

Global Form mode 'readonly'
'''''''''''''''''''''''''''

The form.parameter setting `mode=readonly` will switch the whole form into a `readonly` mode, which is a fast way to use
an existing *Form* just to display the form data, without a possibility for the user to change any data of the form.
The mode can be statically defined in the *Form.parameter* field via::

    mode=readonly

Or dynamically, e.g. via::

    mode={{formModeGlobal:S:alnumx}}

Such variant might be called via SIP. The following shows the same *Form* in the `regular` mode and second in `readonly` mode::

	10.sql = SELECT CONCAT('from&form=person&r=', p.id) as _Pagee, CONCAT('from&form=person&formModeGlobal=readonly&r=', p.id) as _Pagee FROM Person AS p

..

FormElements
------------

* Each *form* contains one or more *FormElement*.
* The *FormElements* are divided in three categories:

  * :ref:`class-container`
  * :ref:`class-native`
  * :ref:`class-action`

* Ordering and grouping: Native *FormElements* and Container-Elements (both with feIdContainer=0) will be ordered by 'ord'.
* Inside of a container, all nested elements will be displayed.
* Technical, it's *not* necessary to configure a FormElement for the primary index column `id`.
* Additional options to a *FormElement* will be configured via the *FormElement.parameter* field (analog to *Form.parameter*
  for *Forms* ).

  * See also: :ref:`comment-space-character`

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

Type: pill
^^^^^^^^^^

* Pill is synonymous for a tab. A pill looks like a tab.
* Pills are only available with mode render='bootstrap'.
* If there is at least one pill defined, every native *FormElement* needs to be assigned to a pill or to a fieldset.
* If there is at least one pill defined, every fieldset needs to be assigned to a pill.

* FormElement settings:

  * *name*: technical name, used as HTML identifier.
  * *label*: Label shown on the corresponding pill button or inside the drop-down menu.
  * *type*: *pill*
  * *feIdContainer*: `0`  - Pill's can't be nested.
  * *parameter*:

    * *maxVisiblePill*: `<nr>` - Number of Pill-Buttons shown. Undefined means unlimited. Excess Pill buttons will be
      displayed as a drop-down menu.

.. _class-native:

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
E.g. if there are upto five elements `grade1, ..., grade5` and the user inputs only the first three, the remaining will be set
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

Class: Native
-------------

Fields:

+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
| Name          | Type                        | Description                                                                                       |
+===============+=============================+===================================================================================================+
| id            | int                         |                                                                                                   |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
| formId        | int                         |                                                                                                   |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|feIdContainer  | int                         |                                                                                                   |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|enabled        | enum('yes'|'no')            |                                                                                                   |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|dynamicUpdate  | enum('yes'|'no')            | In the browser, *FormElements* with "dynamicUpdate='yes'"  will be updated depending on user      |
|               |                             | input. :ref:`dynamic-update`                                                                      |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|name           | string                      |                                                                                                   |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|label          | string                      | Label of *FormElement*. Depending on layout model, left or on top of the *FormElement*            |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|mode           | enum('show', 'readonly',    | *Show*: regular user input field. This is the default.                                            |
|               | 'required',                 | *Required*: User has to specify a value. Typically, an <empty string> represents 'no value'.      |
|               | 'disabled' )                | *Readonly*: user can't change any data. Data not saved.                                           |
|               |                             | *Disabled*: *FormElement* is not visible.                                                         |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|modeSql        | `select` statement with     | A value given here overwrites the setting from `mode`. Most usefull with :ref:`dynamic-update`.   |
|               | a value like in `mode`      | E.g.: {{SELECT IF( '{{otherFunding:FR:alnumx}}'='yes' ,'show', 'hidden' }}                        |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|class          | enum('native', 'action',    | Details below.                                                                                    |
|               | 'container')                |                                                                                                   |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|type           | enum('checkbox', 'date', 'time', 'datetime',  'dateJQW', 'datetimeJQW', 'extra', 'gridJQW', 'text', 'editor', 'note',           |
|               | 'password', 'radio', 'select', 'subrecord', 'upload', 'fieldset', 'pill', 'beforeLoad', 'beforeSave',                           |
|               | 'beforeInsert', 'beforeUpdate', 'beforeDelete', 'afterLoad', 'afterSave', 'afterInsert', 'afterUpdate', 'afterDelete',          |
|               | 'sendMail')                                                                                                                     |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|encode         | 'none', 'specialchar'       | With 'specialchar' (default) the chars <>"'& will be encoded to their htmlentity.                 |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|checkType      | enum('min|max', 'pattern',  |                                                                                                   |
|               | 'number', 'email')          |                                                                                                   |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|checkPattern   | 'regexp'                    |If $checkType=='pattern': pattern to match                                                         |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|onChange       | string                      |List of *FormElement*-names of current form, separated by ', ', If one of the named *FormElements* |
|               |                             | change, reload own data / status / mode                                                           |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|ord            | string                      | Display order of *FormElements* ('order' is a reserved keyword)                                   |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|tabindex       | string                      |HTML tabindex attribute                                                                            |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|size           | string                      |Visible length of input element. Might be ommited, depending on the choosen form layout.           |
|               |                             |Format: <width>,<height> (in characters)                                                           |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|bsLabelColumns | string                      | Number of bootstrap grid columns for label. By default empty, value inherits from the form.       |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|bsInputColumns | string                      | Number of bootstrap grid columns for input. By default empty, value inherits from the form.       |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|bsNoteColumns  | string                      | Number of bootstrap grid columns for note. By default empty, value inherits from the form.        |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|maxLength      | string                      |Maximum characters for input.                                                                      |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|note           | string                      |Note of *FormElement*. Depending on layout model, right or below of the *FormElement*.             |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|tooltip        | text                        |Display this text as tooltip on mouse over.                                                        |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|placeholder    | string                      |Text, displayed inside the input element in light grey.                                            |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|clientJs       | text                        |Javascript called on 'on change' *FormElement*                                                     |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|value          | text                        |Default value                                                                                      |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|sql1           | text                        |SQL query                                                                                          |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|parameter      | text                        |Might contain misc parameter. Depends on the type of *FormElement*.                                |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|feGroup        | string                      | Comma-separated list of Typo3 FE Group ID. NOT SURE IF THIS WILL BE IMPLEMENTED. Native           |
|               |                             | *FormElements*, fieldsets and pills can be assigned to feGroups. Group status: show, hidden,      |
|               |                             | disabled. Group Access: FE-Groups. User will be assigned to FE-Groups and the form definition     |
|               |                             | reference such FE-groups. Easy way of granting permission.                                        |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|deleted        | string                      | 'yes'|'no'.                                                                                       |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|modified       | timestamp                   |updated automatically through stored procedure                                                     |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+
|created        | datetime                    |set once through QFQ                                                                               |
+---------------+-----------------------------+---------------------------------------------------------------------------------------------------+

.. _fe-parameter-attributes:

Attributes defined in the parameter field
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

See also at specific *FormElement* definitions.

+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| Name                   | Type   | Note                                                                                                     |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-pattern-error     | string | Pattern violation: Text for error message used for all FormElements of current form                      |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-required-error    | string | Required violation: Text for error message used for all FormElements of current form                     |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-match-error       | string | Match violation: Text for error message used for all FormElements of current form                        |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| data-error             | string | If none specific is defined: Text for error message used for all FormElements of current form            |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| htmlBefore             | string | HTML Code wrapped before the complete *FormElement*                                                      |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| htmlAfter              | string | HTML Code wrapped after the complete *FormElement*                                                       |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+
| wrapRow                | string | If specified, skip default wrapping (`<div class='col-md-?>`). Instead the given string is used.         |
+------------------------+--------+                                                                                                          |
| wrapInput              | string |                                                                                                          |
+------------------------+--------+                                                                                                          |
| wrapInput              | string |                                                                                                          |
+------------------------+--------+                                                                                                          |
| wrapNote               | string |                                                                                                          |
+------------------------+--------+----------------------------------------------------------------------------------------------------------+

Effect matrix
^^^^^^^^^^^^^

+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| Attribute        | checkbox | dateJQW | datetimeJQW |  gridJQW | extra  | text  | note | password | radio | select | subrecord | timeJQW | upload | editor |
+==================+==========+=========+=============+==========+========+=======+======+==========+=======+========+===========+=========+========+========+
|id                |Internal id                                                                                                                              |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|formId            |Form                                                                                                                            |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|containerId       |Assign the *FormElement* to user defined fieldSet or pill                                                                       |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|enabled           |*FormElement* is active or not                                                                                                  |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|name              |Name of a column of the primary table. *FormElements* with a corresponding table will be saved automatically.                   |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|label             |Label shown to the user.                                                                                                        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|mode              |show, readonly, required, lock, disable.                                                                                        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|class             |native                                                                                                                          |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|type              | checkbox | dateJQW | datetimeJQW |  gridJQW | extra  | text  | note | password | radio | select | subrecord | timeJQW | upload |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|checkType         |          |   -     |   -         |          |        |   -   |      |   -      |       |        |           |   -     |        |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|checkPattern      |          |   -     |   -         |          |        |   -   |      |   -      |       |        |           |   -     |        |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|onChange          |   -      |   -     |   -         |          |        |   -   |      |   -      |   -   |   -    |           |   -     |   -    |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|ord               |   -      |   -     |   -         |   -      |   -    |   -   |   -  |   -      |   -   |   -    |   -       |   -     |   -    |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|tabindex          |   -      |   -     |   -         |   -      |   -    |   -   |   -  |   -      |   -   |   -    |   -       |   -     |   -    |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|size              |   -      |         |             |          |        |   -   |      |   -      |   -   |   -  2 |           |   -     |   -  ? |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|maxLength         |   1      |         |             |          |        |   -   |      |   -      |   - 1 |        |           |         |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|note              |   -      |   -     |   -         |          |        |   -   |   -  |   -      |   -   |   -    |   -       |   -     |   -    |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|tooltip           |   -      |   -     |   -         |          |        |   -   |      |   -      |   -   |   -    |           |   -     |   -    |   ?    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|placeholder       |          |   -     |   -         |          |        |   -   |      |          |       |        |           |   -     |   -    |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|clientJs          |          |   -     |   -         |   -      |        |   -   |      |   -      |   -   |   -    |           |   -     |   -    |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|value             |   -      |   -     |   -         |   -      |   -    |   -   |   -  |   -      |   -   |   -    |           |   -     |   -    |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|sql1              |?                                                                                                                                        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
|*Additional attributes in Field 'parameter'. Typically in key=value format.*                                                                                |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| type             | checkbox | dateJQW | datetimeJQW | gridJQW  | extra  | text  | note | password | radio | select | subrecord | timeJQW | upload | editor |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| accept           |?                                                                                                                                        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| alt              |?                                                                                                                               |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| autocomplete     |          |   -     |   -         |          |        |   -   |      |          |       |        |           |   -     |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| autofocus        |   -      |   -     |   -         |          |        |   -   |      |   -      |   -   |   -    |           |   -     |   -    |   -    |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| checkBoxMode     |   -      |   -     |             |          |        |       |      |          |       |        |           |         |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| checked          |   -      |         |             |          |        |   -   |      |          |   -   |        |           |         |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| unchecked        |   -      |         |             |          |        |   -   |      |          |   -   |        |           |         |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| label2           |   -      |         |             |          |        |       |      |          |   -   |        |           |         |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| itemList         |   -      |         |             |          |        |       |      |          |   -   |   -    |           |         |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| emptyItemAtStart |          |         |             |          |        |       |      |          |       |   -    |           |         |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| emptyItemAtEnd   |          |         |             |          |        |       |      |          |       |   -    |           |         |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| emptyHide        |          |         |             |          |        |       |      |          |       |   -    |           |         |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| retype           |          | -       | -           |          |        | -     |      | -        |       |        |           | -       |        |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| accept           |          |         |             |          |        |       |      |          |       |        |           |         |   -  3 |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| slaveId          |          |         |             |          |        |       |      |          |       |        |           |         |   -    |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| fileDestination  |          |         |             |          |        |       |      |          |       |        |           |         |   -    |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| sqlBefore        |          |         |             |          |        |       |      |          |       |        |           |         |   -    |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| sqlInsert        |          |         |             |          |        |       |      |          |       |        |           |         |   -    |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| sqlDelete        |          |         |             |          |        |       |      |          |       |        |           |         |   -    |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+
| sqlAfter         |          |         |             |          |        |       |      |          |       |        |           |         |   -    |        |
+------------------+----------+---------+-------------+----------+--------+-------+------+----------+-------+--------+-----------+---------+--------+--------+

* 1: A line break created every <size> elements. Easy way to make checkboxes or radio vertical instead of horizontal.
* 2: Any number >1 makes the 'select' input 'multiple' ready.
* See: https://www.w3.org/TR/html5/forms.html#file-upload-state-(type=file)

* All 'native' *FormElements* like 'input', 'checkbox', ...

'autofocus': The first *FormElement* with this attribute will get the focus after form load. If there is no such attribute
 given to any *FormElement*, the attribute will be automatically assigned to the first editable *FormElement*.

To disable 'autofocus' on a form, set 'autofocus=0' on the first editable *FormElement*.

Note: If there are multiple pills defined on a form, only the first pill will be set with 'autofocus'.

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

      {{<columnname>:RZ}}

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

    * *dateFormat*: yyyy-mm-dd | dd.mm.yyyy
    * *showSeconds*: 0|1 - shows the seconds. Independent if the user specifies seconds, they are displayed '1' or not '0'.
    * *showZero*: 0|1 - For an empty timestamp, With '0' nothing is displayed. With '1' the string '0000-00-00 00:00:00' is displayed.

Type: extra
^^^^^^^^^^^

* Element is not shown in the browser.
* The element can be used to define / precalculate values for a column, which do not already exist as a native *FormElement*.
* The element is build / computed on form load and saved alongside with the SIP parameter of the current form.
* Access the value without specifying any store (default store priority is sufficient).

Type: text
^^^^^^^^^^

* General input for text and number.
* *FormElement.size*:

  * <number>:  width of input element in characters. Lineheight = 1.
  * <cols>,<rows>: input element = textarea, width=<cols>, height=<rows>

* *FormElement.parameter*:

  * *retype* = 1 (optional): Current input element will be rendered twice. The form can only submitted if both elements are equal.
  * *retypeLabel* =<text> (optional): The label of the second element.
  * *retypeNote* =<text> (optional): The note of the second element.
  * *characterCountWrap* = <text1>|<text2> (optional). Displays a character counter below the input/textarea element. If
    `text1` / `text2` is missing, just display `<current>/</max>`. Customization: `characterCountWrap=<div class=qfq-cc-style>Count: |</div>`
  * Also check the  :ref:`fe-parameter-attributes` *data-...-error* to customize error messages shown by the validator.


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

LDAP
;;;;

See :ref:`LDAP_Typeahead`

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

* *FormElement.size*:

  * <min_height>,<max_height>: in pixels, including top and bottom bars. E.g.: 300,600


Type: note
^^^^^^^^^^

An FormElement without any 'input' functionality -just to show some text. Use the typical fields 'label', 'value' and
'note' to be displayed in the corresponding three standard columns.

Type: password
^^^^^^^^^^^^^^

* Like a `text` element, but every character is shown as an asterisk.

Type: radio
^^^^^^^^^^^

* Radio Buttons will be built from one of three sources:

  1. 'sql1': E.g. *{{!SELECT type AS label FROM car }}* or *{{!SELECT type AS label, typeNr AS id FROM car}}* or *{{!SHOW tables}}*.

    * Resultset format 'named': column 'label' and optional a column 'id'.
    * Resultset format 'index':

      * One column in resultset >> first column represents *label*
      * Two or more columns in resultset >> first column represents *id* and second column represents *label*.

  2. *FormElement.parameter*:

    * *itemList* = `<attribute>` E.g.: *itemList=red,blue,orange* or *itemList=1:red,2:blue:3:orange*

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
  * *buttonClass*: Instead of the plain radio fields, Bootstrap
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

      {{<columnname>:RZ}}

    For existing records the shown value is as expected the value of the record. For new records, it's the value `0`,
    which is typically not one of the ENUM values and therefore nothing is selected.

Type: select
^^^^^^^^^^^^

* Select lists will be built from one of three sources:

  * 'sql1': E.g. *{{!SELECT type AS label FROM car }}* or *{{!SELECT type AS label, typeNr AS id FROM car}}* or *{{!SHOW tables}}*.

    * Resultset format 'named': column 'label' and optional a column 'id'.
    * Resultset format 'index':

      * One column in resultset >> first column represents *label*
      * Two or more columns in resultset >> first column represents *id* and second column represents *label*.

  * *FormElement.parameter*:

    * *itemList* = `<attribute>` - E.g.: *itemList=red,blue,orange* or *itemList=1:red,2:blue:3:orange*

  * Definition of the *enum* or *set* field (only labels, ids are not possible).

* *FormElement.size*: `<value>`

  * `<value>`: <empty>|0|1: drop-down list.
  * `<value>`: >1: Select field with *size* rows height. Multiple selection of items is possible.

* *FormElement.parameter*:

  * *emptyItemAtStart*: Existence of this item inserts an empty entry at the beginning of the selectlist.
  * *emptyItemAtEnd*: Existence of this item inserts an empty entry at the end of the selectlist.
  * *emptyHide*: Existence of this item hides the empty entry. This is useful for e.g. Enums, which have an empty
    entry and the empty value should not be an option to be selected.

Type: subrecord
^^^^^^^^^^^^^^^

The *FormElement* type 'subrecord' renders a list of records (so called secondary records), typically to show, edit, delete
or add new records. The list is defined as a SQL query. The number of records shown is not limited. These *FormElement*
will be rendered inside the form as a HTML table.

* *sql1*: SQL query to select records. E.g.::

   {{!SELECT a.id AS id, CONCAT(a.street, a.streetnumber) AS a, a.city AS b, a.zip AS c FROM Address AS a}}

  * Notice the **exclamation mark** after '{{' - this is necessary to return an array of elements, instead of a single string.
  * Exactly one column **'id'** has to exist; it specifies the primary record for the target form.
    In case the id should not be visible to the user, it has to be named **'_id'**.
  * Columnname: *[title=]<title>[|[width=]<number>][|nostrip][|icon][|link][|url][|mailto][|_rowClass][|_rowTitle]*

    * All parameter are position independet.
    * Separate parameter by '|'.
    * *[title=]<text>*: Title of the column. The keyword 'title=' is optional. Columns with a title starting with '_' won't be rendered.
    * *[width=]<number>*: Max. width of chars displayed per cell. The keyword 'width=' is optional. Default max width: 20.
      This setting also affects the title of the column.
    * *nostrip*: by default, html tags will be stripped off the cell content before rendering. This protects the table
      layout. 'nostrip' deactivates the cleaning to make pure html possible.
    * *icon*: the cell value contains the name of an icon in *typo3conf/ext/qfq/Resources/Public/icons*. Empty cell values
      will omit an html image tag (=nothing rendered in the cell).
    * *link*: value will be rendered as described under :ref:`column-link`
    * *url*: value will be rendered as a href url.
    * *mailto*: value will be rendered as a href mailto.
    * *_rowClass*

      * The value is a CSS class name(s) which will be rendered in the *<tr class="<_rowClass>">* of the subrecord table.
      * The column itself is hidden to the user.
      * By using Bootstrap, the following predefined classes are available:

        * Text color: *text-muted|text-primary|text-success|text-info|text-warning|text-danger* (http://getbootstrap.com/css/#helper-classes)
        * Row background: *active|success|info|warning|danger* (http://getbootstrap.com/css/#tables-contextual-classes)

    * *_rowTitle*

      * Defines the title attribute of a subrecod table row (tooltip).

    * Examples::

         SELECT note1 AS 'Comment', note2 AS 'Comment|50' , note3 AS 'title=Comment|width=100|nostrip', note4 AS '50|Comment',
         'checked.png' AS 'Status|icon', email AS 'mailto', CONCAT(homepage, '|Homepage') AS 'url',
         ELT(status,'info','warning','danger') AS '_rowClass', help AS '_rowTitle' ...

* *FormElement.parameter*

  * *form*: Target form, e.g. *form=person*
  * *page*: Target page with detail form. If none specified, use the current page.
  * *title*: Title displayed over the table in the current form.
  * *extraDeleteForm*: Optional. The per row delete Button will reference the form specified here (for deleting) instead of the default (*form*).
  * *detail*: Mapping of values from the primary form to the target form (defined via `form=...`).

    * Syntax::

        <source table column name 1|&constant 1>:<target column name 1>[,<source table column name 2|&constant 2>:<target column name 2>][...]

    * Example: *detail=id:personId,&12:xId,&{{a}}:personId*
    * By default, the given value will overwrite values on the target record. In most situations, this is the wished behaviour.
    * Exceptions of the default behaviour have to be defined on the target form in the corresponding *FormElement* in the
      field *value* by changing the default Store priority definition. E.g. `{{<columnname>:RS0}}` - For existing records,
      the store `R` will provide a value. For new records, store `R` is empty and store S will be searched for a value:
      the value defined in `detail` will be choosen. At last the store '0' is defined as a fallback.
    * *source table column name*: E.g. A person form is opened with person.id=5 (r=5). The definition `detail=id:personId`
      and `form=address` maps person.id to address.personId. On the target record, the column personId becomes '5'.
    * *Constant '&'*: Indicate a 'constant' value. E.g. `&12:xId` or `{{...}}` (all possibilities, incl. further SELECT
      statements) might be used.

Type: time
^^^^^^^^^^

* Range time: '00:00:00' to '23:59:59' or '00:00:00'. (http://dev.mysql.com/doc/refman/5.5/en/datetime.html)
* Optional:
* *FormElement.parameter*
  * *showSeconds*: 0|1 - shows the seconds. Independent if the user specifies seconds, they are displayed '1' or not '0'.
  * *showZero*: 0|1 - For an empty timestamp, With '0' nothing is displayed. With '1' the string '00:00[:00]' is displayed.

Type: upload
^^^^^^^^^^^^

* See: https://www.w3.org/TR/html5/forms.html#file-upload-state-(type=file)

An upload element is based on a 'file browse'-button and a 'trash'-button (=delete). Only one of them is shown at a time.
The 'file browse'-button is displayed, if there is no file uploaded already.
The 'trash'-button is displayed, if there is a file uploaded already.

After clicking on the browse brutton , the user can select a file from the local filesystem.
After choosing the file, the upload starts immediately, shown by a turning wheel. When the server received the whole file
and accepts the file, the 'file browse'-button dissappears and the filename is shown, followed by a 'trash'-button.
Either the user is satisfied now or the user can delete the uploaded file (and maybe upload another one).

Until this point, the file is cached on the server but not copied to the `fileDestination`. The user have to save the
current record, either to finalize the upload or to delete a previous uploaded file.

The FormElement behaves like a 'native FormElement' (showing controls/text on the form) as well as an 'action FormElement'
by fireing queries and doing some additional actions during form save. Inside the *Form editor* it's shown as a 'native FormElement'.

* *FormElement.parameter*:

  * *accept*: `image/*,video/*,audio/*,.doc,.docx,.pdf,<mime type>`

  * *fileDestination*: Destination where to copy the file. A good practice is to specify a relative `fileDestination` -
    such an installation (filesystem and database) are moveable.

    * If the original filename should be part of `fileDestination`, the variable *{{filename}}* (STORE_VARS) can be used. Example ::

        fileDestination={{SELECT 'fileadmin/user/pictures/', p.name, '-{{filename}}' FROM Person AS p WHERE p.id={{id:R0}} }}

      * The original filename will be sanatized: only alnum characters are allowed. German 'umlaut' will be replaced by
        'ae', 'ue', 'oe'. All non valid characters will be replaced by '-'.

    * If a file already exist under `fileDestination`, an error message is shown and 'save' is aborted. The user has no
      possibility to overwrite the already existing file. If the whole workflow is correct, this situation should no
      arise. Check also *fileReplace* below.

    * All necessary subdirectories in `fileDestination` are automatically created.

    * Using the current record id in the `fileDestination`: Using {{r}} is problematic for a 'new' primary record: that
      one is still '0' at the time of saving. Use `{{id:R0}}` instead.

  * *slaveId*, *sqlBefore*, *sqlInsert*, *sqlUpdate*, *sqlDelete*, *sqlUpdate*, *sqlAfter*: Only used in :ref:`Upload advanced mode`.

  * *fileReplace=always*: If `fileDestination` exist - replace it by the new one.

Deleting a record and the referenced file
'''''''''''''''''''''''''''''''''''''''''

If the user deletes a record (e.g. pressing the delete button on a form) which contains reference(s) to files, such files
are deleted too. Slave records, which might be also deleted through a 'delete'-form, are *not* checked for file references
and therefore such files are not deleted on the filesystem.

Only columns where the columname contains `pathFileName` are checked for file references. Therefore, always choose a
columnanme which contains `pathFileName`.

If there are other records, which references the same file, such files are not deleted.
It's a very basic check: just the current column of the current table is compared. In general it's not a good idea to
have mutliple references to a single file. Therefore this check is just a fallback.

.. _Upload simple mode:

Upload simple mode
;;;;;;;;;;;;;;;;;;

Requires: *'upload'-FormElement.name = 'column name'* of an column in the primary table.

After moving the file to `fileDestination`, the current record/column will be updated to `fileDestination`.
The database definition of the named column has to be a string variant (varchar, text but not numeric or else).
On form load, the column value will be displayed as path/filename.
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

* *FormElement.value*: The path/filename, shown during 'form load' to indicate a previous uploaded file, has to be queried
  with this field. E.g.::

      {{SELECT pathFilenamePicture FROM Note WHERE id={{slaveId}} }}

* *FormElement.parameter*:

  * *fileDestination*: determine the path/filename. E.g.::

     fileDestination=fileadmin/person/{{name:R0}}_{{id:R}}/uploads/picture_{{filename}}

  * *slaveId*: Defines the target record where to retrieve and store the path/filename of the uploaded file. Check also :ref:`slave-id`. E.g.::

      slaveId={{SELECT id FROM Note WHERE pId={{id:R0}} AND type='picture' LIMIT 1}}

  * *sqlBefore*: fired during a form save, before the following queries are fired.

  * *sqlInsert*: fired if `slaveId=0` and an upload exist (user has choosen a file)::

      sqlInsert={{INSERT INTO Note (pId, type, pathFileName) VALUE ({{id:R0}}, 'image', '{{fileDestination}}') }}

  * *sqlUpdate*: fired if `slaveId>0` and an upload exist (user has choosen a file). E.g.::

      sqlUpdate={{UPDATE Note SET pathFileName = '{{fileDestination}}' WHERE id={{slaveId}} LIMIT 1}}

  * *sqlDelete*: fired if `slaveId>0` and no upload exist (user has not choosen a file). E.g.::

      sqlDelete={{DELETE FROM Note WHERE id={{slaveId:V}}  LIMIT 1}}

  * *sqlAfter*: fired after all previous queries have been fired. Might update the new created id to a primary record. E.g.::

      sqlAfter={{UPDATE Person SET noteIdPicture = {{slaveId}} WHERE id={{id:R0}} LIMIT 1 }}



.. _class-action:

Class: Action
-------------

Type: before... | after...
^^^^^^^^^^^^^^^^^^^^^^^^^^

These type of 'action' *FormElements* will be used to implement data validation or creating/updating additional records.

Types:

  * beforeLoad

    * good to grant access permission.

  * afterLoad
  * beforeSave

    * good to prohibit creating of duplicate records.

  * afterSave

    * good to create & update additional records.

  * beforeInsert
  * afterInsert
  * beforeUpdate
  * afterUpdate
  * beforeDelete
  * afterDelete

sqlValidate
'''''''''''

  Perform checks by fireing a SQL query and expecting a predefined number of selected records.

  * OK: the `expectRecords` number of records has been selected. Continue processing the next *FormElement*.
  * Fail: the `expectRecords` number of records has not been selected (less or more): Display the error message
    `messageFail` and abort the whole (!) current form load or save.

  *FormElement.parameter*:

  * *requiredList* - List of `native`-*FormElement* names: only if all of those elements are filled (!=0 and !=''), the *current*
    `action`-*FormElement* will be processed. This will enable or disable the check, based on the user input! If no
    `native`-*FormElement* names are given, the specified check will always be performed.
  * *sqlValidate* - validation query. E.g.: `sqlValidate={{!SELECT id FROM Person AS p WHERE p.name LIKE {{name:F:all}} AND p.firstname LIKE {{firstname:F:all}} }}`

    * Pay attention to `{{!...` after the equal sign.

  * *expectRecords* - number of expected records.

    * *expectRecords* = `0` or *expectRecords* = `0,1` or *expectRecords* = `{{SELECT COUNT(id) FROM Person}}`
    * Separate multiple valid record numbers by ','. If at least one of those matches, the check will pass successfully.

  * *messageFail* - Message to show. E.g.: *messageFail* = `There is already a person called {{firstname:F:all}} {{name:F:all}}`


.. _slave-id:

slaveId
'''''''

*FormElement.parameter*:

  * *slaveId*:

    * Auto fill: name the action `action`-*FormElement* equal to an existing column (table from the current form definition).
      *slaveId* will be automatically filled with the value of the named column.

      * If there is no such named columnname, set *slaveId* = `0`.

    * Explicit definition: *slaveId* = `123` or *slaveId* = `{{SELECT id ...}}`

Note:

  * `{{slaveId}}` can be used in any query of the current *FormElement*.
  * If the `action`-*FormElement* name exist as a column in the master record: Update that column *automatically* with the
    recent slaveId
  * After an INSERT the `last_insert_id()` becomes the *slaveId*).

sqlBefore / sqlInsert / sqlUpdate / sqlDelete / sqlAfter
''''''''''''''''''''''''''''''''''''''''''''''''''''''''

  * Save values of a form to different record(s), optionally on different table(s).
  * Typically useful on 'afterSave' - be careful when using it earlier, e.g. beforeLoad.

*FormElement.parameter*:

  * *requiredList* - List of `native`-*FormElement*: only if all of those elements are filled, the current
    `action`-*FormElement* will be processed.

  * *sqlBefore*: always fired (before any *sqlInsert*, *sqlUpdate*, ..)
  * *sqlInsert*: fired if *slaveId* == `0` or *slaveId* == `''`.
  * *sqlUpdate*: fired if *slaveId* > `0`.
  * *sqlDelete*: fired if *slaveId* > `0`, after *sqlInsert* or *sqlUpdate*. Be carefull not to delete filled records!
    Always add a check, if values given, not to delete the record! *sqlHonorFormElements* helps to skip such checks.
  * *sqlAfter*: always fired (after *sqlInsert*, *sqlUpdate* or *sqlDelete*).
  * *sqlHonorFormElements*: list of *FormElement* names (this parameter is optional).

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

 * In case of fireing 'sqlInsert', the 'slave.id' of the new created record are copied to master.xId (the database will
     be updated automatically).

 * If the automatic update of the master record is not suitable, the action element should have no name or a name
     which does not exist as a column of the master record. Define  `slaveId={{SELECT id ...}}`

 * Two *FormElements*  `myStreet` and `myCity`:

    * Without *sqlHonorFormElements*. Parameter: ::

       sqlInsert = INSERT INTO address (`street`, `city`) VALUES ('{{myStreet:FE:alnumx:s}}', '{{myCity:FE:alnumx:s}}')
       sqlUpdate = UPDATE address SET `street` = '{{myStreet:FE:alnumx:s}}', `city` = '{{myCity:FE:alnumx:s}}'  WHERE id={{slaveId}} LIMIT 1
       sqlDelete = DELETE FROM address WHERE id={{slaveId}} AND '{{myStreet:FE:alnumx:s}}'='' AND '{{myCity:FE:alnumx:s}}'='' LIMIT 1

    * With *sqlHonorFormElements*. Parameter: ::

       sqlHonorFormElements = myStreet, myCity
       sqlInsert = INSERT INTO address (`street`, `city`) VALUES ('{{myStreet:FE:alnumx:s}}', '{{myCity:FE:alnumx:s}}')
       sqlUpdate = UPDATE address SET `street` = '{{myStreet:FE:alnumx:s}}', `city` = '{{myCity:FE:alnumx:s}}'  WHERE id={{slaveId}} LIMIT 1
       sqlDelete = DELETE FROM address WHERE id={{slaveId}} LIMIT 1

Situation 2: master.id=slave.xId (1:n)

 * Name the action element *different* to any columnname of the master record (or no name).
 * Determine the slaveId: `slaveId={{SELECT id FROM slave WHERE slave.xxx={{...}} LIMIT 1}}`

   * {{slaveId}} == 0 ? 'sqlInsert' will be fired.
   * {{slaveId}} != 0 ? 'sqlUpdate' will be fired.

 * Two *FormElements*  `myStreet` and `myCity`. The `person` is the master record, `address` is the slave:

    * Without *sqlHonorFormElements*. Parameter: ::

       slaveId = SELECT id FROM address WHERE personId={{id}} ORDER BY id LIMIT 1
       sqlInsert = INSERT INTO address (`personId`, `street`, `city`) VALUES ({{id}}, '{{myStreet:FE:alnumx:s}}', '{{myCity:FE:alnumx:s}}')
       sqlUpdate = UPDATE address SET `street` = '{{myStreet:FE:alnumx:s}}', `city` = '{{myCity:FE:alnumx:s}}'  WHERE id={{slaveId}} LIMIT 1
       sqlDelete = DELETE FROM address WHERE id={{slaveId}} AND '{{myStreet:FE:alnumx:s}}'='' AND '{{myCity:FE:alnumx:s}}'='' LIMIT 1

    * With *sqlHonorFormElements*. Parameter: ::

       slaveId = SELECT id FROM address WHERE personId={{id}} ORDER BY id LIMIT 1
       sqlHonorFormElements = myStreet, myCity
       sqlInsert = INSERT INTO address (`personId`, `street`, `city`) VALUES ({{id}}, '{{myStreet:FE:alnumx:s}}', '{{myCity:FE:alnumx:s}}')
       sqlUpdate = UPDATE address SET `street` = '{{myStreet:FE:alnumx:s}}', `city` = '{{myCity:FE:alnumx:s}}'  WHERE id={{slaveId}} LIMIT 1
       sqlDelete = DELETE FROM address WHERE id={{slaveId}} LIMIT 1


Type: sendmail
^^^^^^^^^^^^^^

* Send mail(s) will be processed after:

   * saving the record ,
   * processing all uploads,
   * processing `afterSave` action `FormElements`.


* *FormElement.value*: Body of the email.

* *FormElement.parameter*:

  * *sendMailTo* - Comma-separated list of receiver email addresses. Optional: 'realname <john@doe.com>. If there
    is no recipient email address, *no* mail will be sent.
  * *sendMailCc* - Comma-separated list of receiver email addresses. Optional: 'realname <john@doe.com>.
  * *sendMailBcc* - Comma-separated list of receiver email addresses. Optional: 'realname <john@doe.com>.
  * *sendMailFrom* - Sender of the email. Optional: 'realname <john@doe.com>'. **Mandatory**.
  * *sendMailSubject* - Subject of the email.
  * *sendMailReplyTo* - Reply this email address. Optional: 'realname <john@doe.com>'.
  * *sendMailFlagAutoSubmit* - **on|off** - If 'on' (default), the mail contains the header
    'Auto-Submitted: auto-send' - this suppress a) OoO replies, b) forwarding of emails.
  * *sendMailGrId* - Will be copied to the mailLog record. Helps to setup specific logfile queries.
  * *sendMailXId* - Will be copied to the mailLog record. Helps to setup specific logfile queries.

* To use values of the submitted form, use the STORE_FORM. E.g. `{{name:F:allbut}}`
* To use the `id` of a new created or already existing one, use the STORE_RECORD. E.g. `{{id:R}}`



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

* On **all** `dynamic update` *FormElements* an explicit definition of `value`, including a sanatize class, is necessary
  (except the field is numeric). **A missing definition let's the content overwrite all the time with the old value**.
  A typical definition for `value` looks like::

     {{<FormElement name>::alnumx}}

* Define the receiving *FormElements* in a way, that they will interpret the recent user change! The form variable of the
  specific sender *FormElement* `{{<sender element>:F:<sanitize>}}` should be part of one of the above fields to get an
  impact. E.g.:
  ::

    [receiving *FormElement*].parameter: itemList={{ SELECT IF({{carPriceRange:FE:alnumx}}='expensive','Ferrari,Tesla,Jaguar','General Motors,Honda,Seat,Fiat') }}

  Remember to specify a 'sanatize' class - a missing sanatize class means 'digit', every content, which is not numeric,
  violates the sanatize class and becomes therefore an empty string!


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

* Slave 'interpret' is displayed only for 'pop':

::

   modeSql={{SELECT IF( '{{music:FR:alnumx}}'='pop' ,'show', 'hidden' }}

.. _form-layout:

Form Layout
-----------

The forms will be rendered with Bootstrap CSS classes, based on the 12 column grid model (Bootstrap 3.x).
Generally a 3 column layout for *label* columns on the left side, an *input* field column in the middle and a *note*
column on the right side will be rendered.

The used default column (=bootstrap grid) width is *3,6,3* for *label, input, note*.

* The system wide default can be changed via config.qfq.ini :ref:`config-qfq-ini` - the new settings are the default
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


Best practice
-------------

Central configured values
^^^^^^^^^^^^^^^^^^^^^^^^^

Any variable in *config.qfq.ini* can be used by *{{<varname>:Y}}* in form or report statements.

E.g.

  TECHNICAL_CONTACT = jane.doe@example.net

Could be used in an *FormElement.type* = sendmail with *parameter*  setting *sendMailFrom={{TECHNICAL_CONTACT:Y}}*.

Debug Report
^^^^^^^^^^^^

Writing "report's" in the nested notation or long queries broken over several lines, might not interpreted as wished.
Best for debugging is to specify in the tt-content record::

  debugShowBodyText = 1

Note: Debug information is only display if it's enabled in  *config.ini* by
 * *SHOW_DEBUG_INFO=yes* or
 * *SHOW_DEBUG_INFO=auto* and logged in in the same Browser as a Typo3 backend user.

More detailed error messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If *SHOW_DEBUG_INFO* is enabled, a full stacktrace and variable contents are displayed in case of an error.

Person search form
^^^^^^^^^^^^^^^^^^

QFQ content record::

  # Creates a small form that redirects back to this page
  10 {
    sql = SELECT '_'
    head = <form action='#' method='get'><input type='hidden' name='id' value='{{pageId:T}}'>Search: <input type='text' name='search' value='{{search:CE:all}}'><input type='submit' value='Submit'></form>
  }

  # SQL statement will find and list all the relevant forms
  20 {
    sql = SELECT CONCAT('?detail&form=form&r=', f.id) AS _Pagee, f.id, f.name, f.title
              FROM Form AS f
              WHERE f.name LIKE  '%{{search:CE:all}}%'
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

This example will display grafics instead of text 'add' and 'remove'. Also there is a distance between the templateGroups.

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

* By splitting HTML and JavaScript code over several lines, take care not accidently to create a 'nesting'-end token.
  Check the line after `10.tail =`. It's '}' alone on one line. This is a valid 'nesting'-end token!. There are two options
  to circumvent this:

  * Don't nest the HTML & JavaScript code - bad workaround, this is not human readable.
  * Select different nesting token, e.g. '<' / '>' (check the first line on the following example).

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

       typeAheadSql = SELECT name WHERE name LIKE ? OR firstName LIKE ? LIMIT 100

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

FAQ
---

 * Q: A variable {{<var>}} is shown as empty string, but there should be a value.

   * A: The sanatize rule is violeted and therefore the value has been removed. Set {{<var>:<store>:all}} as a test.
     Only STORE_CLIENT and STORE_FORM will be sanatized.


Report
======

The QFQ extension is activated through tt-content records. One or more tt-content records per page are necessary to render
*forms* and *reports*.

QFQ content element
-------------------

QFQ is used by configuring Typo3 content elements. Insert one or more QFQ content elements on a Typo3 page.
Specify column and language per content record as wished.

The title of the QFQ content element will not be rendered. It's only visible in the backend for orientation.

General
-------

To display a report on any given TYPO3 page, create a content element of type 'QFQ Element' (plugin) on that page.

A simple example
^^^^^^^^^^^^^^^^

Assume that the database has a table person with columns firstName and lastName. To create a simple list of all persons, we can do the following:
::

    10.sql = SELECT id AS pId, CONCAT(firstName, " ", lastName, " ") AS name FROM person

10 Stands for a *root level* of the report (see section `Structure`_). 10.sql defines a SQL query for this specific level. When the query is executed it will return a result having one single column name containing first and last name
separated by a space character.

The HTML output displayed on the page resulting from only this definition could look as follows:
::

    John DoeJane MillerFrank Star

..



I.e., QFQ will simply output the content of the SQL result row after row for each single level.

However, we can modify (wrap) the output by setting the values of various keys for each level: 10.rsep=<br/> for example tells QFQ to seperate the rows of the result by a HTML-line break. The final result in this case is:

::

    10.sql = SELECT id AS personId, CONCAT(firstName, " ", lastName, " ") AS name FROM person
    10.sep = <br>

HTML output:
::

    John Doe<br>Jane Miller<br>Frank Star

..

Syntax
------

    All **root level queries** will be fired in the order specified by 'level' (Integer value).

    For **each** row of a query (this means *all* queries), all subqueries will be fired once.

    *   E.g. if the outer query selects 5 rows, and a nested query always select 3 rows, than the total number of rows are 5 x 3 = 15 rows.

    There is a set of **variables** that will get replaced before the SQL-Query gets executed:

        Column values of the recent rows: {{<level>.<columnname>}}

        Global variables: {{global.<name>}}

        Variables from specific stores: {{<name>[:<store/s>[:<sanitize class>]]}}

        Current row index: {{<level>.line.count}}

        Total rows (num_rows for SELECT and SHOW, affected_rows for UPDATE and INSERT): {{<level>.line.total}}

        Last insert id for INSERT: {{<level>.line.insertId}}

        See :ref:`variables` for a full list of all available variables.

    Be aware that line.count / line.total have to be known before the query is fired. E.g. `10.sql = SELECT {{10.line.count}}, ... WHERE {{10.line.count}} = ...`
    won't work as expected. `{{10.line.count}}` can't be replaced before the query is fired, but will be replaced during processing the result!


    Different types of SQL queries are possible: SELECT, INSERT, UPDATE, DELETE, SHOW

    Only SELECT and SHOW queries will fire subqueries.

    *   Processing of the resulting rows and columns:

    *   In general, all columns of all rows will be printed out sequentially.

        On a per column base, printing of columns can be suppressed. This might be useful to select values which will be
        accessed later on in another query via the {{level.columnname}} variable. To suppress printing of a column, use a
        underscore as column name prefix.

        Reserved column names have a special meaning and will be processed in a special way. See `Processing of columns in the SQL result`_ for details.

        There are extensive ways to wrap columns and rows automatically. See :ref:`wrapping-rows-and-columns`

Debug the bodytext
------------------
The parsed bodytext could be displayed by activating 'showDebugInfo' (:ref:`debug`) and specifying

::

    debugShowBodyText = 1

A small symbol with a tooltip will be shown, where the content record will be displayed on the webpage.
Note: :ref:`debug` information will only be shown with *showDebugInfo=yes* in config.ini .

Structure
---------

A report can be divided into several levels. This can make report definitions more readable because it allows for
splitting of otherwise excessively long SQL queries. For example, if your SQL query on the root level selects a number
of person records from your person table, you can use the SQL query on the second level to look up the city where each person lives.

See the example below:

::

    10.sql = SELECT id AS _pId, CONCAT(firstName, " ", lastName, " ") AS name FROM person
    10.rsep = <br />

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

To make SQL queries, or QFQ records in general, more readable, it's possible to split a line across several lines. Lines
with keywords are on their own (`QFQ Keywords (Bodytext)`_ start a new line). If a line is not a 'keyword' line, it will
be appended to the last keyword line. 'Keyword' lines are detected on:

 * <level>.<keyword> =
 * {
 * <level>[.<level] {

Example::

    10.sql = SELECT 'hello world'
             FROM mastertable
    10.tail = End

    20.sql = SELECT 'a warm welcome'
               'some additional', 'columns'
               FROM smartTable
               WHERE id>100

    20.head = <h3>
    20.tail = </h3>

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

By default, curly braces '{}' are used for nesting. Alternatively angle braces '<>', round braces '()' or square
braces '[]' are also possible. To define the braces to use, the **first line** of the bodytext has to be a comment line and the
last character of that line must be one of '{}[]()<>'. The corresponding braces are used for that QFQ record. E.g.: ::

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

   10.sql = SELECT 'hello world'

   20 {
         sql = SELECT 'a new query'
         head = <h1>
         tail = </h1>
   }

   30 {
         sql = SELECT 'a third query'
         head = <h1>
         tail = </h1>
         40 {
               sql = SELECT 'a nested nested query'
         }
   }

   30.40.tail = End

   50

   {
          sql = SELECT 'A query with braces on their own'
   }


Access to upper column values
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Columns of the upper level result can be accessed via variables, eg. {{10.pId}} will be replaced by the value in the pId column.

+-------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|**Levels**   |A report is divided into levels. Example 1 has 3 levels **10**, **20.25**, **20.25.10**                                                                                                                                      |
+-------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|**Qualifier**|A level is divided into qualifiers **20.30.10** has 3 qualifiers **20**, **30**, **10**                                                                                                                                      |
+-------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|**Root       |Is a level with one qualifier. E.g.: 10                                                                                                                                                                                      |
|levels**     |                                                                                                                                                                                                                             |
+-------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|**Sub        |Is a level with more than one qualifier. E.g. levels **20.25** and **20.30.10**                                                                                                                                              |
|levels**     |                                                                                                                                                                                                                             |
+-------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|**Child**    |The level **20** has one child **20.25**                                                                                                                                                                                     |
+-------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|**Parent**   |The level 20.25 has a parent **20**                                                                                                                                                                                          |
+-------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|**Example    |**10** and **20** are root level and will be executed independently. **10** don't have a sub level. **20.25** will be executed as many times as **20** has row numbers. **20.30.10** won't be executed because there isn't   |
|explanation**|any **20.30** level                                                                                                                                                                                                          |
+-------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


Report Example 1:

::


    # Displays current date
    10.sql = SELECT CURDATE()

    # Show all students from the person table
    20.sql = SELECT p.id AS pId, p.firstName, " - ", p.lastName FROM person AS p WHERE p.typ LIKE "student"

    # Show all the marks from the current student ordered chronological
    20.25.sql = SELECT e.mark FROM exam AS e WHERE e.pId={{20.pId}} ORDER BY e.date

    # This query will never be fired, cause there is no direct parent called 20.30.
    20.30.10.sql = SELECT 'never fired'

.. _wrapping-rows-and-columns:

Wrapping rows and columns: Level keys
-------------------------------------

Order and nesting of queries, will be defined with a typoscript-like syntax: level.sublevel1.subsublevel2. ...
Each 'level' directive needs a final key, e.g: 20.30.10. **sql**. A key **sql** is necessary in order to process a level.
All `QFQ Keywords (Bodytext)`_.

Processing of columns in the SQL result
---------------------------------------

* The content of all columns of all rows will be printed sequentially, without separator.
* Rows with `Special column names`_  will be processed in a special way.

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
|_link                   |Easily create links with different features.                                                                                                                                                 |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_mailto                 |Quickly create email links. A click on the link will open the default mailer. The address is encrypted via JS against email bots.                                                            |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_pageX or _PageX        |Shortcut version of the link interface for fast creation of internal links. The column name is composed of the string *page*/*Page* and a optional character to specify the type of the link.|
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_pdf, _file, _zip       |Shortcut version of the link interface for fast creation of `download`_ links. Used to offer single file download or to concatenate several PDFs and printout of websites to one PDF file.   |
|_Pdf, _File, _Zip       |                                                                                                                                                                                             |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_sendmail               |Send emails.                                                                                                                                                                                 |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_exec                   |Run batch files or executables on the webserver.                                                                                                                                             |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_vertical               |Render Text vertically. This is useful for tables with limited column width.                                                                                                                 |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_img                    |Display images.                                                                                                                                                                              |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_bullet                 |Display a blue/gray/green/pink/red/yellow bullet. If none color specified, show nothing                                                                                                      |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_check                  |Display a blue/gray/green/pink/red/yellow checked sign. If none color specified, show nothing                                                                                                |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|_<nonReservedName>      |Suppress output. Column names with leading underscore are used to select data from the database and make it available in other parts of the report without generating any output.            |
+------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

.. _column-link:

Column: _link
^^^^^^^^^^^^^

{{url | display | **i (internal)**, e(external) | **- (same)**,n (new), p (parent), t(top) | **-**, (e(edit), c(copy), n(new), d(delete), i(insert) , f(file)) }}

* Most URLs will be rendered via class link.
* Column names like `_pagee`, `_mailto`, ... are wrapper to class link.
* The parameters for link contains a prefix to make them position-independet.

+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|URL|IMG|Meaning       |Qualifier                          |Example                    |Description                                                                                                                             |
+===+===+==============+===================================+===========================+========================================================================================================================================+
|x  |   |URL           |u:<url>                            |u:http://www.example.com   |If an image is specified, it will be rendered inside the link, default link class: external                                             |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|x  |   |Mail          |m:<email>                          |m:info@example.com         |Default link class: email                                                                                                               |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|x  |   |Page          |p:<pageId>                         |p:impressum                |Prepend '?' or '?id=', no hostname qualifier (automatically set by browser), default link class: internal, default value: {{pageId}}    |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|x  |   |Download      |d:[<exportFilename>]               |d:complete.pdf             |Link points to `api/download.php`. Additonal parameter are encoded into a SIP. 'Download' needs an enabled SIP.  See `download`_.       |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Text          |t:<text>                           |t:Firstname Lastname       |-                                                                                                                                       |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Render        |r:<mode>                           |r:[0-5]                    |See: `render-mode`_, Default: 0                                                                                                         |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Picture       |P:<filename>                       |P:bullet-red.gif           |Picture '<img src="bullet-red.gif"alt="....">', default link class: internal.                                                           |
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
|   |x  |Bullet        |B:[<color>]                        |B:green                    |Show bullet with '<color>'. Colors: blue, gray, green, pink, red, yellow. Default Color: green.                                         |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |x  |Check         |C:[<color>]                        |C:green                    |Show checked with '<color>'. Colors: blue, gray, green, pink, red, yellow. Default Color: green.                                        |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |URL Params    |U:<key1>=<value1>[&<keyN>=<valueN>]|U:a=value1&b=value2&c=...  |Any number of additional Params. Links to forms: U:form=Person&r=1234                                                                   |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Tooltip       |o:<text>                           |o:More information here    |Tooltip text                                                                                                                            |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Alttext       |a:<text>                           |a:Name of person           |a) Alttext for images, b) Message text for `download`_ popup window.                                                                    |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Class         |c:[n|i|e|<text>]                   |c:i                        |CSS class for link. n:no class attribut, i:internal (ext_localconf.php)(default), e:external (ext_localconf.php), <text>: explicit named|
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
|   |   |File          |f:<filename>                       |f:fileadmin/file.pdf       |Element source for download mode file|pdf|zip. See `download`_.                                                                         |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|   |   |Delete record | x[:a|r|c]                         |x, x:r, x:c                |a: ajax (only QFQ internal used), r: report (default), c: close (current page, open last page)                                          |
+---+---+--------------+-----------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------+

.. _render-mode:

Render mode
^^^^^^^^^^^

+-----------+--------------------+-------------------+----------+-------------------------------------------------------------------+
|Mode       |Both: url & text    |Only: url          |Only: text|Description                                                        |
+===========+====================+===================+==========+===================================================================+
|0 (default)|<a href=url>text</a>|<a href=url>url</a>|          |text or image will be shown, only if there is a url, page or mailto|
+-----------+--------------------+-------------------+----------+-------------------------------------------------------------------+
|1          |<a href=url>text</a>|<a href=url>url</a>|text      |Text or image will be shown, independet of there is a url          |
+-----------+--------------------+-------------------+----------+-------------------------------------------------------------------+
|2          |<a href=url>text</a>|                   |          |no link if text is empty                                           |
+-----------+--------------------+-------------------+----------+-------------------------------------------------------------------+
|3          |text                |url                |text      |no link, only text or image                                        |
+-----------+--------------------+-------------------+----------+-------------------------------------------------------------------+
|4          |url                 |url                |text      |no link, show text, if text is empty, show url                     |
+-----------+--------------------+-------------------+----------+-------------------------------------------------------------------+
|5          |                    |                   |          |nothing at all                                                     |
+-----------+--------------------+-------------------+----------+-------------------------------------------------------------------+


Link Examples
^^^^^^^^^^^^^

+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SQL-Query                                                              |Result                                                                                                                                  |
+=======================================================================+========================================================================================================================================+
|SELECT "m:info@example.com" AS _link                                   |info@example.com as linked text, encrypted with javascript, class=external                                                              |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "m:info@example.com|c:0" AS _link                               |info@example.com as linked text, not encrypted, class=external                                                                          |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "m:info@example.com|P:mail.gif" AS _link                        |info@example.com as linked image mail.gif, encrypted with javascript, class=external                                                    |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "m:info@example.com|P:mail.gif|o:Email" AS _link                |*info@example.com* as linked image mail.gif, encrypted with javascript, class=external, tooltip: "sendmail"                             |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "m:info@example.com|t:mailto:info@example.com|o:Email" AS link  |'mail to *info@example.com*' as linked text, encrypted with javascript, class=external                                                  |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "u:www.example.com" AS _link                                    |www.example as link, class=external                                                                                                     |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "u:http://www.example.com" AS _link                             |*http://www.example* as link, class=external                                                                                            |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "u:www.example.com|q:Please confirm" AS _link                   |www.example as link, class=external, See: `question`_                                                                                   |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "u:www.example.com|c:i" AS _link                                |*http://www.example* as link, class=internal                                                                                            |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "u:www.example.com|c:nicelink" AS _link                         |*http://www.example* as link, class=nicelink                                                                                            |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "p:form_person|c:e" AS _link                                    |<a class="external" href="?form_person">Text</a>                                                                                        |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "p:form_person&note=Text|t:Person" AS _link                     |<a class="internal" href="?form_person&note=Text">Person</a>                                                                            |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "p:form_person|E" AS _link                                      |<a class="internal" href="?form_person"><img alttext="Edit" src="typo3conf/ext/qfq/Resources/Public/icons/edit.gif"></a>                |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "p:form_person|E|g:_blank" AS _link                             |<a target="_blank" class="internal" href="?form_person"><img alttext="Edit" src="typo3conf/ext/qfq/Resources/Public/icons/edit.gif"></a>|
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "p:form_person|C" AS _link                                      |<a class="internal" href="?form_person"><img alttext="Check" src="typo3conf/ext/qfq/Resources/Public/icons/checked-green.gif"></a>      |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "p:form_person|C:green" AS _link                                |<a class="internal" href="?form_person"><img alttext="Check" src="typo3conf/ext/qfq/Resources/Public/icons/checked-green.gif"></a>      |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "U:form=Person&r=123|x|D" as _link                              |<a href="typo3conf/ext/qfq/qfq/api/delete.php?s=badcaffee1234"><span class="glyphicon glyphicon-trash" ></span>"></a>                   |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "U:form=Person&r=123|x|t:Delete" as _link                       |<a href="typo3conf/ext/qfq/qfq/api/delete.php?s=badcaffee1234">Delete</a>                                                               |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+
|SELECT "s:1|d:full.pdf|M:pdf|U:id=det1&r=12|U:id=det2|f:cv.pdf|        |<a href="typo3conf/ext/qfq/qfq/api/download.php?s=badcaffee1234">Download</a>                                                           |
|        t:Download|a:Create complete PDF - please wait" as _link       |                                                                                                                                        |
+-----------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+

.. _question:

Question
^^^^^^^^

**Syntax**

::

    q[:<alert text>[:<level>[:<positive button text>[:<negative button text>[:<timeout>[:<flag modal>]]]]]]


* If a user clicks on a link, an alert is shown. If the user answers the alert by clicking on the 'positive button', the browser opens the specified link.
  If the user click on the negative answer (or waits for timout), the alert is closed and the browser does nothing.
* All parameter are optional.
* Parameter are seperated by ':'
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

.. _download:

Download
^^^^^^^^

Download offers:

* download a single file (any type),
* concatenate several files (uploaded) and/or web pages (=HTML to PDF) into one PDF output file,
* create a ZIP archive, filled with several files ('uploaded' or 'HTML to PDF'-converted).

The downloads are SIP protected. Only the current user can use the link to download files.

By using the `_link` columnname:

* the option `d:...` initiate creating the download link and optional specifies an export filename,
* the optional `M:...` (Mode) specifies the export type (file, pdf, zip),
* setting `s:1` is mandatory for the download function,
* the alttext `a:...` specifies a message in the download popup.

By using `_pdf`,  `_Pdf`, `_file`, `_File`, `_zip`, `_Zip` as columnname, the options `d`, `m` and `s`
will be set by automatically.

All files will be read by PHP - therefore the directory might be protected against direct web access. This way is the
preferred way to offer secure downloads via QFQ.

In case the download needs a persistant URL (no SIP, no user session), a regular
link, pointing directly to a file, have to be used - the download functionality described here is not appropriate for
such a scenario.

.. _download-parameter-files:

Parameter and (element) sources
'''''''''''''''''''''''''''''''

* *download*: `d[:<exportFilename>]`

  * *exportFilename* = <filename for save as> - Name, offered in the 'File save as' browser dialog. Default: 'output.<ext>'.

    If there is no `exportFilename` defined and `mode=file`, than the original filename is taken.

    If the mimetype is different from the `exportFilename` extension, then the mimetype extension will be added to
    `exportFilename`. This guarantees that a filemanager will open the file with the correct application.

    The user typically expect meaningful and distinct filenames for different download links.

* *popupMessage*: `a:<text>` - will be displayed in the popup window during download. If the creating/download is fast,
      the window might disappear quickly.

* *mode*: `m:<mode>`

  * *mode* = <file | pdf | zip> - This parameter is optional and can be skipped in most situations. Mandatory
    for 'zip'.

      * If `m:file`, the mimetype is derived dynamically from the specified file. In this mode, only one element source
        is allowed per download link (no concatenation).

      * In case of multiple element sources, only `pdf` or `zip` is supported.
      * If `m:zip` is used together with `U:...` oder `u:..`, those HTML pages will be converted to PDF. Those files
        get generic filenames inside the archive.
      * If not specified, the **default** 'Mode' depends on the number of specified element sources (=file or web page):

        * If only one `file` is specifed, the default is `file`.
        * If there is a) a page defined or b) multiple elements, the default is `pdf`.

* *element sources* - for `m:pdf` or `m:zip`, all of the following three element sources might be specified multiple times.
     Any combination and order of the three options are allowed.

  * *file*: `f:<pathFilename>` - relative or absolute pathFilename offered for a) download (single), or to be concatenated
            in a PDF or ZIP.
  * *urlParam*: `U:id=<t3 page>&<key 1>=<value 1>&<key 2>=<value 2>&...&<key n>=<value n>`.

    * By default, the options given to wkhtml will *not* be encoded by a SIP!
    * To encode the parameter via SIP: Add '_sip=1' to the URL GET parameter.

      E.g. `U:id=form&_sip=1&form=Person&r=1`.

      In that way, specific sources for the `download` might be SIP encrypted.

    * Any current HTML cookies will be forwarded to/via `wkhtml`. This includes the current FE Login as well as any
      QFQ session. Also the current User-Agent are faked via the `wkhtml` page request.

    * If there are trouble with accessing FE_GROUP protected content, please check `wkhtmltopdf`_.

  * *url*: `u:<url>` - any URL, pointing to an internal or external destination.

  * *WKHTML Options* for `urlParam` or `url`:

    * The 'HTML to PDF' will be done via `wkhtmltopdf`.
    * All possible options, suitable for `wkhtmltopdf`, can be submitted in the `u:...` or `U:...` element source.
      Check `wkhtmltopdf.txt <https://wkhtmltopdf.org/usage/wkhtmltopdf.txt>`_ for possible options. Be aware that
      key/value tuple in the  documentation is separated by a space, but to respect the QFQ key/value notation of URLs,
      the key/value tuple in `u:...` or `U:...` has to be separated by '='. Please see last example below.

	Most of the other Link-Class attributes can be used to customize the link as well.

Example: ::

	# single `file`. Specifying a popup message window text is not necessary, cause a file directly accessed is fast.
	SELECT "d:file.pdf|s|t:Download|f:fileadmin/pdf/test.pdf" AS _link

	# single `file`, with mode
	SELECT "d:file.pdf|m:pdf|s|t:Download|f:fileadmin/pdf/test.pdf" AS _link

	# three sources: two pages and one file
	SELECT "d:complete.pdf|s|t:Complete PDF|U:id=detail&r=1|U:id=detail2&r=1|f:fileadmin/pdf/test.pdf" AS _link AS _link

	# three sources: two pages and one file
	SELECT "d:complete.pdf|s|t:Complete PDF|U:id=detail&r=1|U:id=detail2&r=1|f:fileadmin/pdf/test.pdf" AS _link AS _link

	# three sources: two pages and one file, parameter to wkhtml will be SIP encoded
	SELECT "d:complete.pdf|s|t:Complete PDF|U:id=detail&r=1&_sip=1|U:id=detail2&r=1&_sip=1|f:fileadmin/pdf/test.pdf" AS _link AS _link

	# three sources: two pages and one file, the second page will be in landscape and pagesize A3
	SELECT "d:complete.pdf|s|t:Complete PDF|U:id=detail&r=1|U:id=detail2&r=1&--orientation=Landscape&--page-size=A3|f:fileadmin/pdf/test.pdf" AS _link AS _link

..

Export area
'''''''''''

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

..

Columns: _page[X]
^^^^^^^^^^^^^^^^^

The colum name is composed of the string *page* and a trailing character to specify the type of the link.


**Syntax**

::

    SELECT "[options]" AS _page[<link type>]

    with: [options] = [p:<page & param>]|[t:<text>]|[o:<tooltip>]|[q:<question parameter>]|[c:<class>]|[g:<target>]|[r:<render mode>]

    <link type> = c,d,e,h,i,n,s

..

+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|  column name  |  Purpose                                      |default value of question parameter  |  Mandatory parameters                        |
+===============+===============================================+=====================================+==============================================+
|_page          |Internal link without a grafic                 |empty                                |p:<pageId>[&param]                            |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|_pagec         |Internal link without a grafic, with question  |*Please confirm!*                    |p:<pageId>[&param]                            |
+---------------+-----------------------------------------------+-------------------------------------+----------------------------------------------+
|_paged         |Internal link with delete icon (trash)         |*Delete record ?*                    |U:form=<formname>&r=<record id> *or*          |
|               |                                               |                                     |U:table=<tablename>&r=<record id>             |
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

+-------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|  Parameter  |  Description                                                                                    |  Default value                                           |Example                                                        |
+=============+=================================================================================================+==========================================================+===============================================================+
|<page>       |TYPO3 page id or page alias.                                                                     |The current page: *{{pageId}}*                            |45 application application&N_param1=1045                       |
+-------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<text>       |Text, wrapped by the link. If there is an icon, text will be displayed to the right of it.       |empty string                                              |                                                               |
+-------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<tooltip>    |Text to appear as a ToolTip                                                                      |empty string                                              |                                                               |
+-------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<question>   |If there is a question text given, an alert will be opened. Only if the user clicks on 'ok',     |**Expected "=" to follow "see"**                          |                                                               |
|             |the link will be called                                                                          |                                                          |                                                               |
+-------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<class>      |CSS Class for the <a> tag                                                                        |The default class defined for internal links in           |                                                               |
|             |                                                                                                 |ext_localconf.php (see ...)                               |                                                               |
+-------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<target>     |Parameter for HTML 'target='. F.e.: Opens a new window                                           |empty                                                     |P                                                              |
+-------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<rendermode> |Show/render a link at all or not. See `render-mode`_ 0-5                                         |                                                          |                                                               |
+-------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
|<create sip> |s                                                                                                |                                                          |'s': create a SIP                                              |
+-------------+-------------------------------------------------------------------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+

Column: _paged
^^^^^^^^^^^^^^

These column offers a link, with a confirmation question, to delete one record (mode 'table') or a bunch of records
(mode 'form'). After deleting the record(s), the current page will be reloaded in the browser.

**Syntax**

::

    SELECT "U:table=<tablename>&r=<record id>|q:<question>|..." AS _paged
    SELECT "U:form=<formname>&r=<record id>|q:<question>|..." AS _paged

..

If the record to delete contains column(s), whose columnname match on `%pathFileName%` and such a
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

    SELECT 'U:table=Person&r=123|q:Do you want delete John Doe?' AS _paged
    SELECT 'U:form=person-main&r=123|q:Do you want delete John Doe?' AS _paged

Columns: _Page[X]
^^^^^^^^^^^^^^^^^

* Similar to `_page[X]`
* Parameter are position dependent and therefore without a qualifier!

::

    "[<page id|alias>[&param=value&...]] | [text] | [tooltip] | [question parameter] | [class] | [target] | [render mode]" as _Pagee.

..

Column: _Paged
^^^^^^^^^^^^^^

* Similar to `_paged`
* Parameter are position dependent and therefore without a qualifier!

::

    "[table=<table name>&r-<record id>[&param=value&...] | [text] | [tooltip] | [question parameter] | [class] | [render mode]" as _Paged.
    "[form=<form name>&r-<record id>[&param=value&...] | [text] | [tooltip] | [question parameter] | [class] | [render mode]" as _Paged.

..

Column: _vertical
^^^^^^^^^^^^^^^^^

Render text vertically. This is useful for tables with limited column width. The vertical rendering is achieved via CSS tranformations (rotation) defined in the style attribute of the wrapping tag. You can optionally specify the rotation
angle.

**Syntax**

::

    SELECT "<text>|[<angle>]|[<width>]|[<height>]|[<wrap tag>]" AS _vertical

..

+-------------+-------------------------------------------------------------------------------------------------------+-----------------+
|**Parameter**|**Description**                                                                                        |**Default value**|
+=============+=======================================================================================================+=================+
|<text>       |The string that should be rendered vertically.                                                         |none             |
+-------------+-------------------------------------------------------------------------------------------------------+-----------------+
|<angle>      |How many degrees should the text be rotated? The angle is measured clockwise from baseline of the text.|*270*            |
+-------------+-------------------------------------------------------------------------------------------------------+-----------------+
|<width>      |Width (of what?). Needs to have a CSS_unit (e.g. px, em) specified. (Implemented?)                     |*1em*            |
+-------------+-------------------------------------------------------------------------------------------------------+-----------------+
|<height>     |Height (of what?). Needs to have a CSS-unit (e.g. px, em) specified. (Implemented?)                    |none             |
+-------------+-------------------------------------------------------------------------------------------------------+-----------------+
|<wraptag>    |What tag should be used to wrap the vertical text? Possible options are *div*, *span*, etc.            |*div*            |
+-------------+-------------------------------------------------------------------------------------------------------+-----------------+


**Minimal Example**

::


    10.sql = SELECT "Hallo" AS _vertical

..



**Advanced Examples**

::


    10.sql = SELECT "Hallo|90" AS _vertical
    20.sql = SELECT "Hallo|90|3em|7em|span" AS _vertical

..



Column: _mailto
^^^^^^^^^^^^^^^

Easily create Email links.

**Syntax**

::


    SELECT "<email address>|[<link text>]" AS _mailto

..



+--------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+
|**Parameter** |**Description**                                                                                                                                                                                               |**Default    |
|              |                                                                                                                                                                                                              |value**      |
+==============+==============================================================================================================================================================================================================+=============+
|<emailaddress>|The email address where the link should point to.                                                                                                                                                             |none         |
+--------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+
|<linktext>    |The text that should be displayed on the website and be linked to the email address. This will typically be the name of the recipient. If this parameter is omitted, the email address will be displayed as   |none         |
|              |link text.                                                                                                                                                                                                    |             |
+--------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+


**Minimal Example**

::


    10.sql = SELECT "john.doe@example.com" AS _mailto

..



**Advanced Example**

::


    10.sql = SELECT "john.doe@example.com|John Doe" AS _mailto

..


Column: _sendmail
^^^^^^^^^^^^^^^^^

<TO:email[,email]>|<FROM:email>|<subject>|<body>|[<REPLY-TO:email>]|[<flag autosubmit: on /off>]|[<grid>]|[xId]|<CC:email[,email]>|<BCC:email[,email]>


Send text emails. Every mail will be logged in the table `mailLog`.

**Syntax**

::

    SELECT "john@doe.com|jane@doe.com|Reminder tomorrow|Please dont miss the meeting tomorrow" AS _sendmail

..

+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|**Parameter**                                               |**Description**                                                                           |**Required**|
+============================================================+==========================================================================================+============+
|TO:email[,email]                                            |Comma-separated list of receiver email addresses. Optional: `realname <john@doe.com>`     |    yes     |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|FROM:email                                                  |Sender of the email. Optional: 'realname <john@doe.com>'                                  |    yes     |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|subject                                                     |Subject of the email                                                                      |    yes     |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|body                                                        |Message                                                                                   |    yes     |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|REPLY-TO:email                                              |Email address to reply to (if different from sender)                                      |            |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|flagAutoSubmit  'on' / 'off'                                |If 'on' (default), add mail header 'Auto-Submitted: auto-send' - suppress OoO replies     |            |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|grId                                                        |Will be copied to the mailLog record. Helps to setup specific logfile queries             |            |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|xId                                                         |Will be copied to the mailLog record. Helps to setup specific logfile queries             |            |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|CC:email[,email]                                            |Comma-separated list of receiver email addresses. Optional: 'realname <john@doe.com>'     |            |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+
|BCC:email[,email]                                           |Comma-separated list of receiver email addresses. Optional: 'realname <john@doe.com>'     |            |
+------------------------------------------------------------+------------------------------------------------------------------------------------------+------------+


**Minimal Example**

::


    10.sql = SELECT "john.doe@example.com|company@example.com|Latest News|The new version is now available." AS _sendmail

..



This will send an email with subject *Latest News* from company@example.com to john.doe@example.com.

**Advanced Examples**

::


    10.sql = SELECT "customer1@example.com,Firstname Lastname <customer2@example.com>, Firstname Lastname <customer3@example.com>|company@example.com|Latest News|The new version is now available.|sales@example.com|on|101|222|ceo@example.com|backup@example.com" AS _sendmail

..

This will send an email with subject *Latest News* from company@example.com to customer1, customer2 and customer3 by
using a realname for customer2 and customer3 and suppress generating of OoO answer if any receiver is on vacation.
Additional the CEO as well as backup will receive the mail via CC and BCC.


Column: _img
^^^^^^^^^^^^

Renders images. Allows to define an alternative text and a title attribute for the image. Alternative text and title text are optional.

*   If no alternative text is defined, an empty alt attribute is rendered in the img tag (since this attribute is mandatory in HTML).
*   If no title text is defined, the title attribute will not be rendered at all.

**Syntax**

::


    SELECT "<path to image>|[<alt text>]|[<title text>]" AS _img

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


Column: _pdf | _file | _zip
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most of the other Link-Class attributes can be used to customize the link.
::

    SELECT "[options]" AS _pdf, "[options]" AS _file, "[options]" AS _zip

    with: [options] = [d:<exportFilename]|[U:<params>]|[u:<url>]|[f:file]|[t:<text>]|[a:<message>]|[o:<tooltip>]|[c:<class>]|[r:<render mode>]


* Parameter are position independent.
* *<params>*: see `download-parameter-files`_
* For column `_pdf` and `_zip`, the element sources `U:...`, `u:...`, `f:...` might repeated multiple times.
* Example: ::

		SELECT "f:fileadmin/test.pdf" as _pdf,  "f:fileadmin/test.pdf" as _file,  "f:fileadmin/test.pdf" as _zip
		SELECT "U:id=export&r=1" as _pdf,  "U:id=export&r=1" as _file,  "U:id=export&r=1" as _zip

		SELECT "t:Download PDF|f:fileadmin/test.pdf" as _pdf,  "t:Download PDF|f:fileadmin/test.pdf" as _file,  "t:Download ZIP|f:fileadmin/test.pdf" as _zip
		SELECT "t:Download PDF|U:id=export&r=1" as _pdf,  "t:Download PDF|U:id=export&r=1" as _file,  "t:Download ZIP|U:id=export&r=1" as _zip

		SELECT "d:complete.pdf|t:Download PDF|f:fileadmin/test1.pdf|f:fileadmin/test2.pdf" as _pdf, "d:complete.zip|t:Download ZIP|f:fileadmin/test1.pdf|f:fileadmin/test2.pdf" as _zip

		SELECT "d:complete.pdf|t:Download PDF|f:fileadmin/test.pdf|U:id=export&r=1|u:www.w3c.org" AS _pdf

..

Column: _Pdf | _File | _Zip
^^^^^^^^^^^^^^^^^^^^^^^^^^^

A limited set of attributes is supported: ::

    SELECT "[options]" AS _Pdf, "[options]" AS _File, "[options]" AS _Zip

    with: [options] = [<exportFilename>]|[<text>]|[<1: urlparam|file>]|[<2: urlparam|file>]| ... |[<n: urlparam|file>]

* Parameter are position dependent and therefore without a qualifier!
* `<urlparam>` or `<file>` might be given.
* If there is more than one `<urlparam|file>`, only `_Pdf` or `_Zip` is possible.
* Example: ::

		SELECT "||fileadmin/test.pdf" AS _File, "||fileadmin/test.pdf" AS _Pdf, "||fileadmin/test.pdf" AS _Zip
		SELECT "output.pdf|Download PDF|fileadmin/test.pdf" AS _File, "output.pdf|Download PDF|fileadmin/test.pdf" AS _Pdf, "output.zip|Download ZIP|fileadmin/test.pdf" AS _Zip

		SELECT "complete.pdf|Download PDF|fileadmin/test1.pdf|fileadmin/test2.pdf|id=export&r=1" AS _Pdf

..


Column: _F
^^^^^^^^^^

Challenge 1
'''''''''''

Due to the limitations of MySQL, reserved column names can't be further concatenated. Assume you want to display an image:

::


    # This is valid:
    10.sql = SELECT concat("/static/directory/", p.foto) AS _img FROM person AS p WHERE ...

    # Returns:
    <img src=...>

..



Now assume you want to wrap the image in a div tag:

::


    # This is valid:
    10.sql = SELECT "<div>", CONCAT("/static/directory/", p.foto) AS _img, "</div>" FROM person AS p WHERE ...

    # Returns:
    <div><img src=...></div>

..



The example above works fine - however, as soon as you want to use *field wrappers*, things get messy:

::


    # This is valid:
    10.sql = SELECT "<div>", CONCAT("/static/directory/", p.foto) AS _img, "</div>" FROM person AS p WHERE ...
    10.fbeg = <td>
    10.fend = </td>

    # Returns:
    <td><div></td><td><img src=...></td><td></div></td>

..



To achieve the desired result, one might want to try something like this:

::


    # This is NOT valid:
    10.sql = SELECT CONCAT("<div>", concat("/static/directory/", p.foto) AS _img, "</div>") FROM person AS p WHERE ...
    10.fbeg = <td>
    10.fend = </td>

    # Returns a MySQL error because nesting concat() -functions is not allowed

..



Challenge 2
'''''''''''

Assume you have multiple columns with reserved names in the same query and want to use one of them in a later query:

::


    10.sql = SELECT CONCAT("/static/directory/", g.picture) AS _img, CONCAT("/static/preview/", g.thumbnail) AS _img FROM gallery AS g WHERE ...

    20.sql = SELECT "{{10.img}}", d.text FROM description AS d ...

..



The example above will fail because there are two img columns which can not be distinguished.

Solution
''''''''

The reserved column 'F'(=Format) can be used to

*   further wrap columns with a reserved name

*   assign an arbitrary name to a column built through a reserved name to make it accessible in later queries.

Solution for *#Challenge_1*:

::


    10.sql = SELECT CONCAT("Q:img|T:div") AS wrappedImg FROM person AS p WHERE ...
    10.fbeg = <td>
    10.fend = </td>

    # Returns:
    <td><div><img src=...></div></td>

..



Solution for *#Challenge_2*:

::


    10.sql = SELECT CONCAT("Q:img|V:mypic") AS wrappedImg FROM person AS p WHERE ...

    20.sql = SELECT "{{10.mypic}}" ...

..



+-------------+--------------------------------------------------------------------+--------+
|**Parameter**|**Description**                                                     |Required|
+=============+====================================================================+========+
|Q            |Any of the *reserved column names*                                  |        |
+-------------+--------------------------------------------------------------------+--------+
|Z            |Process the column but don't display it                             |        |
+-------------+--------------------------------------------------------------------+--------+
|X            |Strip tags / Remove all tags                                        |        |
+-------------+--------------------------------------------------------------------+--------+
|T            |Wrap the column with the defined tag. F.e.: T:tdcolspan="2"         |        |
+-------------+--------------------------------------------------------------------+--------+
|V            |Define an unambiguous variable name for this colum. F.e.: V:someName|        |
+-------------+--------------------------------------------------------------------+--------+
|*            |Add all the parameters required for the column defined with Q:      |        |
+-------------+--------------------------------------------------------------------+--------+


The above example builds a link to pageB - refer to the :ref:`column-link`-manual for details. The link tells page B to
render the form with name formname and load the record with id id for editing.

QFQ CSS Classes
---------------

* `qfq-table-50`, `qfq-table-80` - release the default width of 100% and specify minwidth=50% resp. 80%.

* Background Color: `qfq-color-grey-1`, `qfq-color-grey-2`  (table, row, cell)
* Table: `table`
* Table > hover: `table-hover`
* Table > condensed: `table-condensed`

E.g.::

  10.sql = SELECT id, name, firstName, ...
  10.head = <table class='table table-condensed qfq-table-50'>

Examples
--------

The following section gives some examples of typical reports

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


    10.sql = SELECT "Hello World<br />"
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

Formating (i.e. wrapping of data with HTML tags etc.) can be achieved in two different ways:

One can add formatting output directly into the SQL by either putting it in a separate column of the output or by using
concat to concatenate data and formatting output in a single column.

One can use ?level keys to define formatting information that will be put before/after/between all rows/columns of the
actual levels result.

Two columns

::


    # Add the formating information as a coloum
    10.sql = SELECT p.firstName, " " , p.lastName, "'<br /'>" FROM exp_person AS p

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


    10.sql = SELECT p.name FROM exp_person AS p
    10.rend = <br />

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

Result:

::


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
    10.rend = <br />
    20.sql = SELECT a.street FROM exp_address AS a
    20.rend = <br />

Two queries: nested ::

    # outer query
    10.sql = SELECT p.name FROM exp_person AS p
    10.rend = <br />

    # inner query
    10.10.sql = SELECT a.street FROM exp_address AS a
    10.10.rend = <br />

* For every record of '10', all records of 10.10 will be printed.

Two queries: nested with variables ::

    # outer query
    10.sql = SELECT p.id, p.name FROM exp_person AS p
    10.rend = <br />

    # inner query
    10.10.sql = SELECT a.street FROM exp_address AS a WHERE a.pId='{{10.id}}'
    10.10.rend = <br />

* For every record of '10', all assigned records of 10.10 will be printed.

Two queries: nested with hidden variables in a table ::

    10.sql = SELECT p.id AS _pId, p.name FROM exp_person AS p
    10.rend = <br />

    # inner query
    10.10.sql = SELECT a.street FROM exp_address AS a WHERE a.pId='{{10.pId}}'
    10.10.rend = <br />

Same as above, but written in the nested notation ::

  10 {
    sql = SELECT p.id AS _pId, p.name FROM exp_person AS p
    rend = <br />

    10 {
    # inner query
      sql = SELECT a.street FROM exp_address AS a WHERE a.pId='{{10.pId}}'
      rend = <br />
    }
  }

* Columns starting with a '_' won't be printed but can be accessed as regular columns.
