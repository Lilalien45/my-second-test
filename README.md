<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>URL List</title>

<style>
body {
    font-family: Arial, sans-serif;
    margin: 20px;
}
button {
    display: block;
    width: 80px;
    margin: 4px 0;
    padding: 6px;
    cursor: pointer;
}
</style>

<script>
function redirectToPage(number) {
    window.location.href = "https://lilalien45.github.io/my-second-test/test-" + number + ".html";
}

window.onload = function() {
    const container = document.getElementById("links");

    for (let i = 1; i <= 200; i++) {
        const button = document.createElement("button");
        button.textContent = "Page " + i;
        button.onclick = function() {
            redirectToPage(i);
        };
        container.appendChild(button);
    }
};
</script>
</head>
<body>
<h2>Select a Page</h2>
<div id="links"></div>
</body>
</html>
