# Как быстро переключаться между кластерами kubernetes ?
Если у вас больше одного kubernetes кластера у разных провайдеров, вам постоянно приходится переключаться между ними. 
Приходится помнить форматы настроики контекста для всех правайдеров которые вы используете.

Решение этой проблемы утилита *kubeconfig-loader* которую написал David Steiman(https://github.com/xetys). 

Давате настроим эту утилиту и насладимся ее удобством.

## Скачиваем и устанавливаем:
```
curl https://raw.githubusercontent.com/xetys/kubeconfig-loader/master/kubeconfig-loader > kubeconfig-loader
chmod kubeconfig-loader
mv kubeconfig-loader /usr/local/bin
```
## Как пользоваться:
Добавлем кластер
```
kubeconfig-loader add azure-prj ~/.kube/config
```
Убеждаемся что он добавлен. Эта команда покажет список конфигураций. 
```
kubeconfig-loader ls
```
Переключаемся на следующий кластер. Наприем для переключения на hetzner надо выполнить следующую команду:
```
hetzner-kube cluster kubeconfig starship
```
Добавлвем этот кластер в пул. 
```
kubeconfig-loader add hetzner-starship ~/.kube/config
```
Если мы сейчас запросим список конфигураций то увидим в ней две конфигурации: azure-prj и hetzner-starship. Переключаемся быстро и просто:
```
kubeconfig-loader use azure-prj
```
Преключение происходит гараздо быстрее чем с помощью нативных утилит. Если контекст вам больше не нужен удалите его с помощью команды:
```
kubeconfig-loader use azure-prj
```

Удачи в настройке !

