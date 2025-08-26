# AWS CodeBuild – Buildspec (Python)

## Referência

* [Documentação Oficial do Buildspec](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)

---

## Estrutura do **buildspec.yaml**

```yaml
version: 0.2

phases:
  install:        # Instalação de dependências e setup do ambiente
    runtime-versions:
      python: 3.9
    commands:
      - echo "Instalando dependências Python..."
      - pip install --upgrade pip
      - pip install -r requirements.txt

  pre_build:      # Passos antes do build, como testes unitários
    commands:
      - echo "Executando testes..."
      - pytest --maxfail=1 --disable-warnings -q

  build:          # Fase principal de build
    commands:
      - echo "Rodando aplicação / empacotando projeto"
      - python setup.py sdist bdist_wheel || echo "Sem setup.py, apenas rodando script"

  post_build:     # Etapas finais (artefatos, validações)
    commands:
      - echo "Build concluído em `date`"
      - ls -lh

artifacts:        # Arquivos que serão exportados como resultado
  files:
    - '**/*'
  discard-paths: no
```

---

## Exemplo 1 – Aplicação Python Simples

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.9
    commands:
      - echo "Instalando dependências"
      - pip install -r requirements.txt

  build:
    commands:
      - echo "Rodando aplicação Python"
      - python main.py

artifacts:
  files:
    - '**/*'
```

---

## Exemplo 2 – Python + Testes Unitários

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.10
    commands:
      - echo "Instalando pacotes necessários"
      - pip install -r requirements.txt
      - pip install pytest

  pre_build:
    commands:
      - echo "Executando testes unitários"
      - pytest tests/ --junitxml=reports/tests.xml

  build:
    commands:
      - echo "Empacotando projeto"
      - python setup.py sdist

artifacts:
  files:
    - dist/**/*
    - reports/tests.xml
```

---

## Exemplo 3 – Python + Docker (ECR Deploy)

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo "Logando no Amazon ECR..."
      - $(aws ecr get-login --no-include-email --region $AWS_DEFAULT_REGION)
      - REPOSITORY_URI=<AWS_ACCOUNT_ID>.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/meu-repo-python
      - IMAGE_TAG=$(date +%Y%m%d%H%M%S)
  build:
    commands:
      - echo "Build da imagem Docker"
      - docker build -t $REPOSITORY_URI:$IMAGE_TAG .
  post_build:
    commands:
      - echo "Fazendo push da imagem"
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo "Gerando imagedefinitions.json para deploy"
      - printf '[{"name":"python-app","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

Esse último é útil quando o Python roda dentro de container e o deploy acontece no **ECS** ou **EKS**.
