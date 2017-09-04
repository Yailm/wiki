
input[type=file] clear
------------------------------------

    var f = document.getElementById("file");
        if (f.value) {
            try{
                f.value = ''; //for IE11, latest Chrome/Firefox/Opera...
            } catch (err) {}
        if (f.value) { //for IE5 ~ IE10
            var form = document.createElement('form'), ref = f.nextSibling, p = f.parentNode;
            form.appendChild(f);
            form.reset();
            p.insertBefore(f,ref);
        }
    }
