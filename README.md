<!DOCTYPE html>
<html>
<head>
    <title>Redirector</title>
</head>
<body>
    <script>
        // This script runs inside the HTML to find the number in your URL
        const currentUrl = window.location.href;
        
        // This finds the number between "test-" and ".html"
        const match = currentUrl.match(/test-(\d+)\.html/);
        
        if (match) {
            const currentNumber = parseInt(match[1]);
            const nextNumber = currentNumber + 1;

            if (nextNumber <= 200) {
                // Change the number and go to the next page
                const nextUrl = currentUrl.replace(`test-${currentNumber}`, `test-${nextNumber}`);
                window.location.href = nextUrl;
            } else {
                document.body.innerHTML = "<h1>Finished! Reached test-200.</h1>";
            }
        } else {
            // If it's the very first page (test.html), go to test-2.html
            window.location.href = currentUrl.replace("test.html", "test-2.html");
        }
    </script>
    <p>Redirecting to the next number...</p>
</body>
</html>
