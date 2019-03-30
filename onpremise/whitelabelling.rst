Whitelabelling
==============

SentinelTrails allows on-premise installations to be whitelabelled. To use whitelabeling:

* Place directory named ``config`` in the dir where you start the application from.
* Place file named ``application.properties`` inside config dir
* The following properties can be overridden:

    * ``styling.dir`` - path to the dir where logo and css files stay
    * ``styling.footer`` - can contain html (it's not escaped)
    * ``styling.logo`` - name of file inside <styling.dir>; if <styling.dir> is not defined this can be url
    * ``styling.css`` - name of file inside <styling.dir>; if <styling.dir> is not defined this can be url
    * ``styling.title`` - page title
    * ``styling.login.name`` - name on login page
    * ``styling.login.footer`` - footer on login page ;can contain html (it's not escaped)

Example:

.. code:: text

	styling.dir=file:config/whitelabel/
	styling.footer=&copy; 2018
	styling.logo=your_logo.jpg
	styling.css=your_styles.css
	styling.title=White Label
	styling.login.name=Company name
	styling.login.footer=&copy; 2019
