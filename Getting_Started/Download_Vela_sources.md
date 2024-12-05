# Download openvela sources

\[ English | [简体中文](Download_Vela_sources_zh-cn.md) \]

The openvela source is located in a collection of Git repositories hosted by [GitHub](https://github.com/open-Vela) or [Gitee](https://gitee.com/open-vela). 

## Initialize the Repo client

1. Create and navigate to a working directory

    ```
    mkdir vela-opensource
    cd vela-opensource
    ```

2. Initialize your working directory for source control:

    - Github(need to register a public key, please refer to [Github documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)):

        ```
        # Install Git LFS (Large File Storage) for managing large files
        sudo apt install git-lfs
        git lfs install
        git lfs --version

        repo init --partial-clone -u git@github.com:open-vela/manifests.git -b dev -m openvela.xml --git-lfs
        ```

    - gitee(need to register a public key, please refer to [Gitee documentation](https://gitee.com/help/articles/4191)):

        ```
        # Install Git LFS (Large File Storage) for managing large files
        sudo apt install git-lfs
        git lfs install
        git lfs --version
        
        repo init --partial-clone -u git@gitee.com:open-vela/manifests.git -b dev -m openvela.xml --git-lfs
        ```

## Download the openvela source

Run the following command to download the openvela source tree to working directory:

```
repo sync -c -j$(nproc)
```

## What to do next
Please refer to [Build openvela from sources](./Build_Vela_from_sources.md).
