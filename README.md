# Github workflow

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

## Adding secret key
https://docs.github.com/en/actions/using-workflows/about-workflows#advanced-workflow-features
