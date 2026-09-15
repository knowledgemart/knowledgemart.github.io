---
title: Extract data from GitHub using Python and Airflow
date: 2023-01-31 00:00:00 +0000 
tags: [Python, Airflow]
---

## PyGithub intro

### Installation and access token

PyGithub is a Python library to interact with the GitHub API. It allows developers to access and manipulate GitHub resources such as repositories, issues, pull requests, and users. To install PyGithub, you can use the following pip command in your terminal or command prompt: `pip install PyGithub`.

To import the library, execute the following in the Python script: `from github import Github`.

Before you start writing the code, you need to generate a personal access token to interact with the API. 

1. Go to the GitHub website and sign in to your account.
2. Click on your profile picture in the top-right corner and select Settings.
3. In the left sidebar, click on Developer settings.
4. Under Developer settings, click on Personal access tokens.
5. Click on Generate new token.
6. Give your token a descriptive name and select the permissions you need (see below).
7. Click on Generate token.

I recommend the following minimum permissions, to begin with:

`read:user` – to access your account details.

`repo` – to interact with public and private repositories.

You will only be able to see the generated token once, so make sure to save it in a safe place. You can use this token to authenticate your API requests.

### Basic classes and methods
`Github` class is the main class to interact with the GitHub API. It takes the personal access token as its argument.


```python
# create an instance of the Github class and authenticate with a token
g = Github('')  # paste your access token here
print(g)
```

    <github.MainClass.Github object at 0x78a8030cfb50>

<br>
`get_user()` method returns the authenticated Github user.

```python
my_user = g.get_user()

print(my_user)
print('Login:', my_user.login)
print('Name:', my_user.name)
print('Location:', my_user.location)
print('Bio:', my_user.bio)
print('Number of public repos:', my_user.public_repos)
```

    AuthenticatedUser(login=None)
    Login: thedarksidepy
    Name: None
    Location: None
    Bio: Welcome to the Dark Side
    Number of public repos: 1

<br>
`get_repo()` method returns a specific repository by its name and owner.

```python
my_repo = g.get_repo('thedarksidepy/thedarksidepy.github.io')

print(my_repo)
print('Name:', my_repo.name)
print('Description:', my_repo.description)
print('Language:', my_repo.language)
print('URL:', my_repo.html_url)
```

    Repository(full_name="thedarksidepy/thedarksidepy.github.io")
    
    Name: thedarksidepy.github.io
    Description: None
    Language: Shell
    URL: https://github.com/thedarksidepy/thedarksidepy.github.io

<br>
`get_user().get_repos()` method returns a list of repositories owned by the authenticated user.

```python
my_repos = g.get_user().get_repos()

print(my_repos)
print('\n')

for repo in my_repos:
    if repo.owner.login == my_user.login:  # filter out repositories of my organization 
        print('Name:', repo.name)
        print('Description:', repo.description)
        print('Language:', repo.language)
        print('URL:', repo.html_url)
        print('\n')
```

	    <github.PaginatedList.PaginatedList object at 0x78a7e2d25610>

      Name: thedarksidepy.github.io
      Description: None
      Language: Shell
      URL: https://github.com/thedarksidepy/thedarksidepy.github.io


<br>
`get_commits()` method of the repository object returns a list of commits.

```python
commits = g.get_repo('thedarksidepy/thedarksidepy.github.io').get_commits()

print(commits)
print('\n')

for commit in commits:
    print('SHA:', commit.sha)
    print('Author:', commit.author.name)
    print('Date:', commit.commit.author.date)
    print('Message:', commit.commit.message)
    print('\n')

```

	  <github.PaginatedList.PaginatedList object at 0x78a7e2d6c510>


    SHA: 2a974f4c409bb4d4d995279e9735e252a1160c88
    Author: None
    Date: 2025-07-02 16:47:31+00:00
    Message: Update 2023-04-30-chat-gpt-and-data-engineering.md
    
    
    SHA: c7de4f847a7d9568e4919597b123f7c01f96faf3
    Author: None
    Date: 2025-07-02 16:47:13+00:00
    Message: Create authors.yml
    
    
    SHA: a431560a9135c69cdb1c1fc58fb914734f353a02
    Author: None
    Date: 2025-07-02 16:45:58+00:00
    Message: Update 2023-01-31-extract-data-github-python-airflow.md
    
    
    SHA: 8a7d46ab66e5f61125d49b2d203155a16c921120
    Author: None
    Date: 2025-07-02 16:40:05+00:00
    Message: Add files via upload
    
    (...)
	

<br>
The `get_contents()` method in PyGithub is used to retrieve the contents of a file or directory in a GitHub repository. This method returns a `ContentFile` object for a file or a list of `ContentFile` objects for a directory. Each `ContentFile` object has information about the file, including the type (file or directory), the name, the path, and the content.

```python
my_repo = g.get_repo('thedarksidepy/thedarksidepy.github.io')

sample_file = my_repo.get_contents('.gitmodules')

print('Type:', sample_file.type)
print('Name:', sample_file.name)
print('Path:', sample_file.path)
print('\n')
print('Content:', sample_file.decoded_content)
```

    Type: file
    Name: .gitmodules
    Path: .gitmodules
    
    Content: b'[submodule "assets/lib"]\n\tpath = assets/lib\n\turl = https://github.com/cotes2020/chirpy-static-assets.git\n'
	
<br>	

```python
my_repo = g.get_repo('thedarksidepy/thedarksidepy.github.io')

sample_dir = my_repo.get_contents('assets')

for content in sample_dir:
    print('Type:', content.type)
    print('Name:', content.name)
    print('Path:', content.path)
    print('\n')
```

      Type: dir
      Name: img
      Path: assets/img
      
      Type: file
      Name: lib
      Path: assets/lib

<br>    	
## GithubOperator in Airflow {#githuboperator-in-airflow}
#### Installation and setup {#installation-and-setup}

You can use all PyGithub methods inside Airflow's GithubOperator.

First, install the `apache-airflow-providers-github` library.

Next, create a GitHub connection.

1. Go to the Airflow UI.
2. Click on the Admin tab at the top of the page.
3. Click on the Connections link on the left sidebar.
4. Click on the Create button on the right side of the page.
5. In the `Conn Id` field, enter `github_default`.
6. In the `Conn Type` field, select `GitHub`.
7. Fill in the remaining fields with the appropriate information for your GitHub account, such as the `Login` and `Password` or `Token`.
8. Click on the Save button to create the connection.

![](/assets/img/2023-01-31-extract-github-data/Screenshot 2023-01-25 at 12.04.10.png)

Similarly in Airflow 3, except you have to provide the access token in the `Password` field.

![](/assets/img/2023-01-31-extract-github-data/Screenshot 2025-07-02 at 21.17.10.png)

You should now see `github_default` listed under the Connections page and you can use it in your DAGs. 

Import the GithubOperator as follows:  
`from airflow.providers.github.operators.github import GithubOperator`

<br>
### Get user data {#get-user-data}
The code below defines an instance of the `GithubOperator` class. This specific instance performs the `get_user` method with no arguments. The result of the API call is then processed using a lambda function that returns a formatted string containing information about the GitHub user. 

```python
get_user_info = GithubOperator(
    task_id='get_user_info',
    github_method='get_user',
    github_method_args={},
    result_processor=lambda user: f'''Login: {user.login}, 
                                      Name: {user.name}''',
)
```

![](/assets/img/2023-01-31-extract-github-data/Screenshot 2025-07-02 at 21.23.02.png)

<br>
### Get repo data {#get-repo-data}
Similarly to the example above, you can retrieve information about a repository. The result of the API call is processed using a lambda function that returns a formatted string containing information about the GitHub repository. Note that the `get_repo()` method requires a `full_name_or_id` argument.

```python
get_repo_info = GithubOperator(
    task_id='get_repo_info',
    github_method='get_repo',
    github_method_args={'full_name_or_id': 
                        'thedarksidepy/thedarksidepy.github.io'},
    result_processor=lambda repo: f'''Name: {repo.name}, 
                                      Description: {repo.description}''',
)
```

![](/assets/img/2023-01-31-extract-github-data/Screenshot 2025-07-02 at 21.24.36.png)

<br>
### List repositories {#list-repositories}
Let's return a list of all repositories' names where the currently authenticated user is the owner. 

```python
list_repos = GithubOperator(
    task_id='list_repos',
    github_method='get_user',
    github_method_args={},
    result_processor=lambda user: 
                     [repo.name for repo in user.get_repos()
                      if repo.owner.login == user.login],
)
```

![](/assets/img/2023-01-31-extract-github-data/Screenshot 2025-07-02 at 21.26.20.png)

Naturally, you can retrieve all the repo details as before. Let's return a list of dictionaries. Each dictionary in the list contains the name, description, programming language, and URL of a repository. The repositories are filtered so that only those owned by the authenticated user are included in the list.

```python
list_repos_details = GithubOperator(
    task_id='list_repos_details',
    github_method='get_user',
    github_method_args={},
    result_processor=lambda user: 
                    [dict(name=repo.name, 
                          description=repo.description,
                          language=repo.language,
                          URL=repo.html_url)
                      for repo in user.get_repos()
                      if repo.owner.login == user.login]
)
```

![](/assets/img/2023-01-31-extract-github-data/Screenshot 2025-07-02 at 21.26.45.png)

<br>
### List commits {#list-commits}
The below piece of code returns commits details. Each dictionary in the list contains the SHA, author name, commit date, and commit message for a single commit. The processed result is then returned by the lambda function and can be used in downstream tasks in the workflow.

```python
list_commits = GithubOperator(
    task_id='list_commits',
    github_method='get_repo',
    github_method_args={'full_name_or_id': 
                        'thedarksidepy/thedarksidepy.github.io'},
    result_processor=lambda repo: 
                    [dict(SHA=commit.sha, 
                          author=commit.author.name,
                          date=str(commit.commit.author.date),
                          message=commit.commit.message)
                      for commit in repo.get_commits()]
)
```

![](/assets/img/2023-01-31-extract-github-data/Screenshot 2025-07-02 at 21.27.06.png)

<br>
### Get contents {#get-contents}
Lastly, let's retrieve the contents of the `assets` directory.

```python
get_contents = GithubOperator(
    task_id='get_contents',
    github_method='get_repo',
    github_method_args={'full_name_or_id': 
                        'thedarksidepy/thedarksidepy.github.io'},
    result_processor=lambda repo: 
                    [dict(type=content.type, 
                          name=content.name,
                          path=content.path)
                      for content in repo.get_contents('assets')]
)
```

![](/assets/img/2023-01-31-extract-github-data/Screenshot 2025-07-02 at 21.27.16.png)


<br>
## Hands-on example: list all file paths in a repository {#hands-on-example--list-all-file-paths-in-a-repository}
The below piece of code uses the PyGithub library to retrieve information about the contents of a GitHub repository.

The first line retrieves a `Repository` object for the repository with the full name `thedarksidepy/thedarksidepy.github.io` using the `get_repo` method of a `Github` object.

The second line retrieves the contents of the root directory of the repository using the `get_contents` method of the `Repository` object and stores them in the `contents` variable.

The rest of the code implements a while loop that uses `pop` and `extend` methods to traverse the contents of the repository and construct a list of file paths, stored in the `files_list` variable. The loop continues until the contents list is empty or has only one item. The loop checks the type of each content item, using the `type` attribute. If it's a directory, it retrieves the contents of the directory using the `get_contents` method of the `Repository` object and appends them to the contents list. If it's a file, its path is appended to the `files_list` list. The resulting `files_list` variable contains a list of file paths for all the files in the repository.
 
```python
repo = g.get_repo('thedarksidepy/thedarksidepy.github.io')
contents = repo.get_contents('')
	
files_list = []
	
while len(contents) > 1:
    file_content = contents.pop(0)
    if file_content.type == 'dir':
        contents.extend(repo.get_contents(file_content.path))
    else:
        files_list.append(file_content.path)
        
print(files_list)
```

	['.editorconfig', '.gitattributes', '.gitignore', '.gitmodules', '.nojekyll', 'Gemfile', 'LICENSE', 'README.md', '_config.yml', 'index.html', '.devcontainer/devcontainer.json', '.devcontainer/post-create.sh', '.vscode/extensions.json', '.vscode/settings.json', '.vscode/tasks.json', '_data/authors.yml', '_data/contact.yml', '_data/share.yml', 
	(...)

<br>
The same can be implemented in Airflow. First, you need to define a Python function as follows.

```python
def list_paths(repo, contents):
    files_list = []
	
    while len(contents) > 1:
        file_content = contents.pop(0)
        if file_content.type == 'dir':
            contents.extend(repo.get_contents(file_content.path))
        else:
            files_list.append(file_content.path)

    return files_list
```

Then call it inside the `GithubOperator` task. The `result_processor` argument is set to a lambda function that calls the `list_paths` function with the `repo` and `repo.get_contents('')` as arguments.

```python
list_paths_task = GithubOperator(
    task_id='list_paths_task',
    github_method='get_repo',
    github_method_args={'full_name_or_id': 
                        'thedarksidepy/thedarksidepy.github.io'},
    result_processor=lambda repo: list_paths(repo, repo.get_contents(''))
)

```
