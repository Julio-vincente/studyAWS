# X-RAY
O AWS X-Ray é uma serviço que permite rastrear solicitações de sua aplicação feitas em aplicações distrubuídas, facilitando a identificação de problemas na infraestrutura com o tempo de cada requisição. Ele é mais utilizado dentro de aplicações de microsserviços.

### Exemplo X-Ray Python
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Ativa o X-Ray monitorar bibliotecas populares automaticamente
patch_all()
```

## X-Ray MiddleWare
O MiddleWare que se integra com requisições HTTP dentro da aplicação, pode ser usado com Flask, Django, etc.

### Exemplo Flask
```python
from flask import Flask
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
from aws_xray_sdk.core import xray_recorder

app = Flask(__name__)

# Configura o nome do serviço que vai aparecer no X-Ray
xray_recorder.configure(service='Aplicação')

# Habilita o middleware do X-Ray no Flask
XRayMiddleware(app, xray_recorder)
```
Esse exemplo faz com que todas as rotas do Flask passem a ser monitoradas pelo X-Ray.

### Exemplo 2 Flask
```python
from flask import Flask
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
from aws_xray_sdk.core import xray_recorder

app = Flask(__name__)
xray_recorder.configure(service='Aplicação')
XRayMiddleware(app, xray_recorder)

# Rota Default Flask
@app.route("/processar")
def processar():
    # Cria um subsegmento com um nome personalizado
    with xray_recorder.in_subsegment('simulando-processamento') as subsegment:
        time.sleep(1.5) # Simula um processamento
        subsegment.put_metadata('dado_exemplo', {'info': 'valor importante'}, 'custom')
    
    return "OK!!"
```
