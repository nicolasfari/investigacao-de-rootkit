# investigacao-de-rootkit
# Sneaky Patch — TryHackMe Writeup

## Descrição do Lab
Um sistema de alto valor foi comprometido. Analistas de segurança detectaram atividades suspeitas no kernel, mas a presença do invasor permanece oculta. As ferramentas de detecção tradicionais falharam e o intruso estabeleceu persistência profunda no sistema.

**Objetivo:** Investigar e identificar uma porta dos fundos em nível de kernel e encontrar a Flag.

---

## Conceitos Envolvidos
- Rootkit de kernel (Kernel-mode rootkit)
- Módulos de kernel Linux (.ko)
- Análise de /proc e /sys
- Extração de strings de binários
- Decodificação hexadecimal

---

## Metodologia

### 1. Listando módulos do kernel
O primeiro passo foi verificar os módulos carregados no kernel:

```bash
lsmod
```
<img width="718" height="793" alt="image" src="https://github.com/user-attachments/assets/aa57159f-8c7e-4b28-8084-f257eca883c3" />

**Resultado suspeito: spatch**
**Sinais de alerta:**
- spatch — nenhuma função legítima conhecida
- Tamanho pequeno — 12288 bytes
- Used by 0 — nenhum módulo depende dele

---

### 2. Confirmando ocultação via /proc/modules

```bash
cat /proc/modules | grep spatch
```

**Resultado:**
<img width="982" height="616" alt="image" src="https://github.com/user-attachments/assets/95aaf107-2409-460d-be00-691f97e035d5" />

**Dois sinais críticos:**
- `0x0000000000000000` — endereço de memória zerado. Módulo legítimo nunca tem endereço zero o rootkit está manipulando o kernel para esconder seu endereço real
- `OE` — Out-of-tree (O) e module taint (E) módulo não faz parte do kernel oficial

---

### 3. Verificando conexões de rede

```bash
ss -tulnp
```
**Observação:** Porta 80 aberta sem processo identificado — o rootkit estava ocultando o processo responsável.

---

### 4. Localizando o arquivo do módulo

```bash
find / -name "spatch.ko" 2>/dev/null
```

<img width="996" height="881" alt="image" src="https://github.com/user-attachments/assets/b11f244c-0754-4672-91d6-9b882125ae58" />

---

### 5. Extraindo strings do módulo

```bash
strings /usr/lib/modules/6.8.0-1016-aws/kernel/drivers/misc/spatch.ko
```

**Resultado relevante:**
<img width="996" height="881" alt="image" src="https://github.com/user-attachments/assets/ca9a369b-8d82-4f6e-8969-8591e910e7d8" />

A flag estava codificada em hexadecimal dentro do binário do módulo.

---

### 6. Decodificando a flag

Usando CyberChef (From Hex) ou o comando:

```bash
echo "54484d7b73757033725f736e33346b795f643030727d0a" | xxd -r -p
```

**Flag:** `THM{sup3r_sn34ky_d00r}`
<img width="1792" height="692" alt="image" src="https://github.com/user-attachments/assets/d35b0443-605c-4d51-9c71-988b511e882c" />

---

## Lições Aprendidas

- Rootkits de kernel se instalam como módulos `.ko` para ter acesso total ao sistema operacional
- Endereço de memória zerado em `/proc/modules` é sinal claro de rootkit ocultando sua localização real
- Ferramentas tradicionais como `ps` e `netstat` podem ser enganadas por rootkits de kernel sempre compare múltiplas fontes
- O comando `strings` é uma forma simples e eficaz de extrair informações de binários suspeitos
- Decodificação hex é uma habilidade básica essencial para análise forense

---

## Ferramentas Utilizadas
- `lsmod` — listar módulos do kernel
- `cat /proc/modules` — verificar ocultação
- `ss -tulnp` — verificar conexões de rede
- `find` — localizar arquivos suspeitos
- `strings` — extrair strings de binários
- `xxd` — decodificar hexadecimal
- CyberChef — https://gchq.github.io/CyberChef

---

## Referências
- TryHackMe: https://tryhackme.com
- MITRE ATT&CK T1547.006 — Kernel Modules and Extensions:
https://attack.mitre.org/techniques/T1547/006/

