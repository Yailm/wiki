
eclipse disable GTK+3
------------------------------------

* with the SWT_GTK3=0 environment variable when you start eclipse

        $ SWT_GTK3=0 eclipse

* another option is to add the following line to `/usr/lib/eclipse/eclipse.ini`

        --launcher.GTK_version
        2

    Those two lines must be added **before**, and remember to install `webkitgtk2` for javadoc pop up

        --launcher.appendVmargs
