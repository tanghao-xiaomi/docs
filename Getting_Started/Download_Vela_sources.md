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

    - For Github:

        ```
        # Install Git LFS (Large File Storage) for managing large files
        sudo apt install git-lfs
        git lfs install
        git lfs --version

        repo init --partial-clone -u git@github.com:open-vela/manifests.git -b dev -m openvela.xml --git-lfs
        ```

    - For gitee:

        ```
        repo init --partial-clone -b opensource -m vela.xml -u {GITEE_URL}
        ```

## Download the openvela source

Run the following command to download the openvela source tree to working directory:

```
repo sync -c -j8
```

## What to do next
Please refer to [Build openvela from sources](./Build_Vela_from_sources.md).
