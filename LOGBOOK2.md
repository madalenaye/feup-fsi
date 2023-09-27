
# Trabalho realizado nas Semanas #2 e #3

## Identificação

- A vulnerabilidade CVE-2023-23392 refere-se a uma falha nos sistemas operativos da Microsoft: Windows 11 Version 21H2, 22H2 e Server 2022
- Esta falha permite a execução remota de um pacote enviado para um servidor, afetando a Pilha do Protocolo HTTP
- O atacante que envia o pacote malicioso não tem de estar autenticado no servidor

## Catalogação

- A vulnerabilidade foi registada no Microsoft Security Response Center (MSRC) e no National Institute of Standards and Technology (NIST)
- A falha foi reportada no dia 14 de março de 2023 por um autor desconhecido
- Pese embora a Microsoft tenha um programa de bug-bounty, não se sabe se este reporting se inseriu nele
- O nível de gravidade foi caracterizado como crítico: 9.8 na escala de severidade Common Vulnerability Scoring System (CVSS)

## Exploit

- Até ao momento, não se conhece nenhum exploit ativo ou público
- Atualmente, estima-se que o preço de um exploit ronde os 10/20 mil dólares
- No dia-0, o custo do exploit encontrar-se-ia no intervalo de 50 mil a 100 mil dólares
- O CTI Interest Score associado à vulnerabilidade é 0.21, o que pode explicar a ausência de exploits conhecidos

## Ataques

- Apesar de não existirem ataques reais conhecidos, se estes ocorrerem, existe muito potencial para causar danos
- O atacante pode correr código malicioso no servidor, comprometendo a proteção do mesmo
- A confidencialidade e a integridade dos recursos alojados no servidor são afetadas

## Fontes

- https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-23392
- https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-23392
- https://nvd.nist.gov/vuln/detail/CVE-2023-23392
- https://vuldb.com/?id.223027