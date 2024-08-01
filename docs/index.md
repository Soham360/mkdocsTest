# Documentation Dashboard

<div class="dropdown-container">
    <select id="repository-dropdown" onchange="populateFileDropdown()">
        <option value="">Select a Repository</option>
        <option value="PPODLinkML">PPODLinkML</option>
        <option value="hello_icicle_auth_clients">hello_icicle_auth_clients</option>
        <option value="galyleo">galyleo</option>
        <option value="basic_skills">basic_skills</option>
        <!-- Add more repositories here -->
    </select>
    <select id="file-dropdown" onchange="displayFileContent()">
        <option value="">Select a YAML file</option>
        <!-- Options will be populated based on the selected repository -->
    </select>
</div>

## File Content

<div id="file-content"></div>

<style>
    body {
        font-family: Arial, sans-serif;
        margin: 20px;
        max-width: 90%;
        margin-right: auto;
    }
    
    .dropdown-container {
        display: flex;
        gap: 10px;
        margin: 10px 0;
    }
    
    #repository-dropdown, #file-dropdown {
        padding: 10px;
        flex: 1;
    }

    #file-content pre {
        background-color: #f5f5f5;
        padding: 10px;
        border: 1px solid #ddd;
        overflow-x: auto;
        font-size: 12px; /* Smaller text */
        white-space: pre-wrap; /* Wrap long lines */
    }
</style>

<script>
    const repoPaths = {
        "PPODLinkML": "ICICLE-ai/PPODLinkML",
        "hello_icicle_auth_clients": "ICICLE-ai/hello_icicle_auth_clients",
        "galyleo": "Soham360/galyleo",
        "basic_skills": "Soham360/basic_skills"
        // Add GitHub paths for more repositories if needed
    };

    const githubApiUrl = "https://api.github.com/repos/";

    async function populateFileDropdown() {
        const repoDropdown = document.getElementById('repository-dropdown');
        const fileDropdown = document.getElementById('file-dropdown');
        const selectedRepo = repoDropdown.value;

        // Clear previous options
        fileDropdown.innerHTML = '<option value="">Select a YAML file</option>';

        if (selectedRepo) {
            const files = await fetchYAMLFiles(selectedRepo);
            files.forEach(file => {
                const option = document.createElement('option');
                option.value = file;
                option.textContent = file;
                fileDropdown.appendChild(option);
            });
        }
    }

    async function fetchYAMLFiles(repoName) {
        const response = await fetch(`${githubApiUrl}${repoPaths[repoName]}/contents/`);
        const data = await response.json();
        let files = [];
        await Promise.all(data.map(async (item) => {
            if (item.type === 'dir') {
                files = files.concat(await fetchFilesRecursively(item.url));
            } else if (item.name.endsWith('.yaml') || item.name.endsWith('.yml')) {
                files.push(item.path);
            }
        }));
        return files;
    }

    async function fetchFilesRecursively(dirUrl) {
        const response = await fetch(dirUrl);
        const data = await response.json();
        let files = [];
        await Promise.all(data.map(async (item) => {
            if (item.type === 'dir') {
                files = files.concat(await fetchFilesRecursively(item.url));
            } else if (item.name.endsWith('.yaml') || item.name.endsWith('.yml')) {
                files.push(item.path);
            }
        }));
        return files;
    }

    async function displayFileContent() {
        const repoDropdown = document.getElementById('repository-dropdown');
        const fileDropdown = document.getElementById('file-dropdown');
        const fileContentDiv = document.getElementById('file-content');

        const selectedRepo = repoDropdown.value;
        const selectedFile = fileDropdown.value;

        if (selectedRepo && selectedFile) {
            const response = await fetch(`https://raw.githubusercontent.com/${repoPaths[selectedRepo]}/main/${selectedFile}`);
            const content = await response.text();
            fileContentDiv.innerHTML = `<pre>${content}</pre>`;
        } else {
            fileContentDiv.innerHTML = '';
        }
    }
</script>
