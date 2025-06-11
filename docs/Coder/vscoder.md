# Coder

## Referências

* [ Coder Server ]( https://github.com/coder/code-server )
* [ Script coder ]( https://github.com/Julio-vincente/bootStrap/blob/master/coderPage.sh )

---

## Instalação

```bash
curl -fsSL https://code-server.dev/install.sh | sh
sudo systemctl enable --now code-server@$USER
```

Dentro do arquivo config.yaml configure para o seu servidor
```bash
vim /home/ubuntu/.config/code-server/config.yaml

bind-addr: 0.0.0.0:8080 # escutando todas as interfaces
auth: password
password: '$PASSWORD' # variavel de uma senha
cert: false
```

Validação
```bash
sudo systemctl restart code-server@ubuntu
sudo systemctl status code-server@ubuntu
```
