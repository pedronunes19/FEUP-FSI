# SEED Labs - SQL Injection Attack Lab

## Task 1: Posting a Malicious Message to Display an Alert Window

## Task 2: Posting a Malicious Message to Display Cookies

## Task 3: Stealing Cookies from the Victim’s Machine

## Task 4: Becoming the Victim’s Friend


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



## Desafio 2
