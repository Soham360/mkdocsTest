<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Repository Files</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        select {
            margin-right: 10px;
        }
        #file-list {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <h1>Repository YAML Files</h1>
    
    <label for="repo-select">Select Repository:</label>
    <select id="repo-select">
        <!-- Options will be dynamically added here -->
    </select>
    
    <div id="file-list">
        <!-- YAML files will be listed here -->
    </div>

    <script>
        const repositories = {
            "hello_icicle_auth_clients": "https://api.github.com/repos/ICICLE-ai/hello_icicle_auth_clients/contents",
            // Add more repositories as needed
        };

        const repoSelect = document.getElementById('repo-select');
        const fileList = document.getElementById('file-list');

        // Populate the dropdown with repositories
        Object.keys(repositories).forEach(repo => {
            const option = document.createElement('option');
            option.value = repositories[repo];
            option.textContent = repo;
            repoSelect.appendChild(option);
        });

        // Recursively fetch files from a directory
        async function fetchFiles(url) {
            let files = [];
            let directories = [url];

            while (directories.length > 0) {
                const currentUrl = directories.pop();
                try {
                    const response = await fetch(currentUrl);
                    const contents = await response.json();

                    for (const item of contents) {
                        if (item.type === 'file' && (item.name.endsWith('.yaml') || item.name.endsWith('.yml'))) {
                            files.push(item);
                        } else if (item.type === 'dir') {
                            directories.push(item.url);
                        }
                    }
                } catch (error) {
                    console.error('Error:', error);
                }
            }
            return files;
        }

        // Event listener for dropdown change
        repoSelect.addEventListener('change', async (event) => {
            const repoUrl = event.target.value;
            fileList.innerHTML = 'Loading...';

            try {
                const files = await fetchFiles(repoUrl);

                fileList.innerHTML = '';
                if (files.length > 0) {
                    files.forEach(file => {
                        const fileLink = document.createElement('a');
                        fileLink.href = file.download_url;
                        fileLink.textContent = file.name;
                        fileLink.target = '_blank';
                        fileLink.style.display = 'block';

                        fileList.appendChild(fileLink);
                    });
                } else {
                    fileList.innerHTML = 'No YAML files found.';
                }

            } catch (error) {
                fileList.innerHTML = 'Error loading files.';
                console.error('Error:', error);
            }
        });

        // Trigger change event to load the first repo's files
        repoSelect.dispatchEvent(new Event('change'));
    </script>
</body>
</html>
