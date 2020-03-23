# hello world python library

## How to create python library

1. Create `setup.py`
1. Create directory with module name, `hello_module`
1. Put `__init__.py` and `hello_world.py`
1. `python setup.py bdist_egg` or `python setup.py bdist_whl`
1. Directory

    ```
    ± tree
    .
    ├── README.md
    ├── build
    │   ├── bdist.macosx-10.13-x86_64
    │   └── lib
    │       └── hello_module
    │           ├── __init__.py
    │           └── hello_world.py
    ├── dist
    │   ├── hello_module-0.1-py3.6.egg
    │   ├── hello_module-0.2-py3-none-any.whl
    │   └── hello_module-0.2-py3.6.egg
    ├── hello_module
    │   ├── __init__.py
    │   └── hello_world.py
    ├── hello_module.egg-info
    │   ├── PKG-INFO
    │   ├── SOURCES.txt
    │   ├── dependency_links.txt
    │   └── top_level.txt
    └── setup.py

    7 directories, 13 files
    ```

## How to use it in Glue Job (python shell)

1. Upload to s3

    ```
    aws s3 cp dist/hello_module-0.2-py3-none-any.whl s3://<your_bucket>/naka/lib/hello_world/
    upload: dist/hello_module-0.2-py3-none-any.whl to s3://<your_bucket>/naka/lib/hello_world/hello_module-0.2-py3-none-any.whl
    ```

1. Create Job script

    ```hello_world_glue_job.py
    from hello_module import hello_world

    hello_world.hello_world()
    ```

1. Create job with library with Terraform

    ```
    resource "aws_glue_job" "hello-world" {
      name         = "hello-world"
      glue_version = "1.0"
      max_capacity = 0.0625
      max_retries  = 0
      role_arn     = "arn:aws:iam::<account-name>:role/service-role/AWSGlueServiceRole-test"
      timeout      = 2880

      default_arguments = {
        "--job-language"   = "python"
        "--extra-py-files" = "s3://<your_bucket>/naka/lib/hello_world/hello_module-0.2-py3-none-any.whl"
      }

      command {
        name            = "pythonshell"
        python_version  = "3"
        script_location = "s3://<your_bucket>/glue/script/hello_world_glue_job.py"
      }

      execution_property {
        max_concurrent_runs = 1
      }
    }
    ```
