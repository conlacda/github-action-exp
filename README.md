# Github workflow
[![learn-github-actions](https://github.com/conlacda/github-action-exp/actions/workflows/learn-github-actions.yml/badge.svg)](https://github.com/conlacda/github-action-exp/actions/workflows/learn-github-actions.yml)
## Tham khảo
* [Github action repo](https://github.com/actions) - tại đây chứa các môi trường khác nhau để tự setup.
## Cấu trúc cho 1 github workflow

Tạo file `.github/workflows/ii_job_name.yml`

```yml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]

jobs:
  job-with-github-action-repo: # tên của job
    runs-on: ubuntu-latest # môi trường chạy
    steps:
      - name: This is the first step name
        uses: actions/checkout@v3
      - uses: actions/setup-node@v3 # không có tên thì bỏ qua name
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v # cmd
      - run: ./example.sh # chạy 1 file có trong source code
        shell: bash
```
Đoạn này rất quan trọng:
```yml
    steps:
      - name: This is the first step name
        uses: actions/checkout@v3
```
thể hiện việc dùng action của github. Nếu không có thì nhiều phần không hoạt động (ví dụ: chạy 1 file bash, lấy danh sách file,...)
## Sử dụng env

`env` có thể được sử dụng tại nhiều nơi như bên dưới
```yml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]
env:
  DAY_OF_WEEK: Monday
jobs:
  greeting-job:
    runs-on: ubuntu-latest
    env:
      Greeting: Hello
    steps:
      - name: "Say Hello Mona it's Monday"
        if: ${{ env.DAY_OF_WEEK == 'Monday' }}
        run: echo "$Greeting ${{ env.First_Name }}. Today is $DAY_OF_WEEK!"
        env:
          First_Name: Mona
```
`env` sẽ được gom thành 1 object.  
**Define**
```yml
env:
  name: Long
  age: 18
```
**Usage**
```yml
$name
$age
${{ env.name }}
```

## If-else
```yml
env:
  name: Long
jobs:
  steps:
    - name: if else job
      if: ${{ env.name == 'Long'}}
      run: echo "name is Long"
```
Không có `else`, muốn dùng else thì viết thêm **!** vào

## Chạy file bash
Tạo file `example.sh` nằm ở thư  mục ngoài
```sh
echo "Hello from a sh file"
```

<details>
  <summary>Github action run a bash file</summary>

```yml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]

jobs:
  job-with-github-action-repo: # tên của job
    runs-on: ubuntu-latest # môi trường chạy
    steps:
      - name: This is the first step name
        uses: actions/checkout@v3
      - run: sh ./example.sh
```
</details>

## Run
```yml
- run: pwd
  shell: bash

- run: |
    echo "run multiple lines"
```

## Secret key
> https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository

Tab `Setting` -> `Secrets and variables` -> `Actions` -> `Repository secrets` tạo 1 key với `name = SECRET_KEYY` và `content = sk`
<details>
  <summary>Secret key example</summary>

```yml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]

jobs:
  job-with-github-action-repo: # tên của job
    runs-on: ubuntu-latest # môi trường chạy
    steps:
      - name: This is the first step name
        uses: actions/checkout@v3
      - name: Retrieve secret
        env:
          super_secret: ${{ secrets.SECRET_KEYY }}
        if: ${{ env.super_secret == 'sk'}} 
        run: echo "sk is skk"
```
</details>

Tại action, mọi từ khóa trùng với key sẽ được chuyển thành dấu `**`. Tại ví dụ này `echo "sk is skk"` sẽ thành `*** is ***k`.  
Thế nên secret key lúc nào cũng phải là 1 chuỗi dài và đảm bảo nó ko thể xuất hiện trong file `yml` 1 chút nào. Ví dụ key 15 kí tự random,..

## Dependent jobs
Mặc định các job được chạy song song với nhau. Nếu muốn chạy tuần tự thì dùng **needs**

<details>
  <summary>Github dependent jobs</summary>

  ![image](https://user-images.githubusercontent.com/33364412/212527545-f3600d1f-ab4b-4a47-bd1a-c47bd5f430ae.png)
```yml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - run: echo "setting up"
  build:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - run: echo "building app"
  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "testing"
```
</details>

## Matrix
Khi muốn chạy cùng 1 việc trên nhiều phiên bản thì dùng matrix. Ví dụ muốn test trên c++11, c++14, c++17, c++20 cùng 1 lúc.

<details>
  <summary>Run with matrix</summary>
  
  ![image](https://user-images.githubusercontent.com/33364412/212527696-e77fbe22-27d2-45c9-8451-8b28abfc17ae.png)

```yml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [12, 14, 16]
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
```
</details>

## Chạy test mỗi khi push

<details>
  <summary>Python test file</summary>

```py
# content of test_sample.py
def func(x):
    return x + 1


def test_answer():
    assert func(3) == 5

# run "pytest" to test.
# pip install pytest
```
</details>

<details>
  <summary>Github action run test on push</summary>

```yml
# https://github.com/marketplace/actions/setup-python
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Python
        uses: actions/setup-python@v4.5.0
        with:
          python-version: '3.10' 
      - run: |
          pip install pytest
          pytest

```
</details>

## Github action on specific branches
<details>
  <summary>Expand example</summary>

```yml
on:  
  push:
    branches: # multiple branches
      - main
      - dev
  pull_request:
    branches:
      - main
  pull_request_comment: # chỉ với dòng này mà không có thêm branches lồng bên trong nghĩa là chạy trên mọi nhánh

on: [push] # run on every push

```
</details>


## Tạo badge
Tại màn hình run job có luôn phần tạo badge

## Check lint
TODO: check lint mỗi khi push code

### Thử nghiệm auto-deploy với github action | tích hợp deploy với netlify, vercel | tích hợp Circle CI vào github action

## Thêm ghi chú github context
${{ secrets.GITHUB_TOKEN }}

## Github action with docker

## Github action with other CI/CD services