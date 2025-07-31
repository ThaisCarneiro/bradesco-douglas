# Atualização do AAP para 2.5 – Remoção da versão 2.4 do EDA

Após realizar em laboratório a atualização do AAP para **2.5**, o único ponto de atenção foi com relação à **VM do EDA**, já que se pretende utilizá-la novamente.

Abaixo os passos que segui para remover a instalação do **EDA 2.4** antes de executar o script de instalação do AAP 2.5:

**Referência:** https://access.redhat.com/solutions/7101637

---

## Etapas executadas - Na VM do EDA

```bash
# 1. Parar os serviços
systemctl stop automation-eda-controller nginx.service

#2. Listar pacotes instalados
dnf list installed | grep automation-eda

#3. Remover pacotes
dnf remove automation-eda-controller automation-eda-controller-server automation-eda-controller-ui

#4. Remover arquivos
rm -rf /var/lib/ansible-automation-platform
rm -rf /etc/ansible-automation-platform

#5. Reiniciar
reboot 


#########################
#Mesmo após as etapas acima, no meu ambiente persistia alguns links simbólicos que precisei remover na mão. 
#A minha instalação falhava com a mensagem "Failed to enable unit: Unit file automation-eda-controller-worker.target does not exist."
########################

#6. Verificar dependencias do servico. Notar que existe o "automation-eda-controller-worker.target" na lista
systemctl list-dependencies --plain automation-eda-controller.target

#7. Verificar links remanescentes
ls -lR /etc/systemd/system/ | grep automation-eda-controller-worker
#output:
#lrwxrwxrwx. 1 root root 63 May 27 14:59 automation-eda-controller-worker.target -> /usr/lib/systemd/system/automation-eda-controller-worker.target
#lrwxrwxrwx. 1 root root 63 May 27 14:59 automation-eda-controller-worker.target -> /usr/lib/systemd/system/automation-eda-controller-worker.target

#8. Encontrar os links
find /etc/systemd/system/ -name 'automation-eda-controller-worker.target'
#output:
#/etc/systemd/system/multi-user.target.wants/automation-eda-controller-worker.target
#/etc/systemd/system/automation-eda-controller.target.wants/automation-eda-controller-worker.target

#9. Remover links
unlink /etc/systemd/system/multi-user.target.wants/automation-eda-controller-worker.target
unlink /etc/systemd/system/automation-eda-controller.target.wants/automation-eda-controller-worker.target

#10. reiniciar systemd
systemctl daemon-reexec
systemctl daemon-reload
```

Fora isso, não esquecer de remover a base atual e criar o schema novamente antes de rodar o setup.sh:
```bash
DROP DATABASE automationedacontroller;
CREATE DATABASE automationedacontroller;
```
