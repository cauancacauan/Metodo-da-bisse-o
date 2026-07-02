# Metodo-da-bisse-o
# Utilizado para encontrar raizes em uma função.


import numpy as np
import warnings

def criar_funcao_segura(expressao):
    """
    Transforma a string digitada pelo usuário em uma função matemática real.
    Isso é feito de forma segura usando numpy para que o usuário possa digitar
    coisas como 'sin(x)' ou 'x**2' sem riscos de segurança.
    """
    # Dicionário com as operações matemáticas permitidas (para segurança)
    operacoes_permitidas = {
        'x': None,  # Será substituído pelo valor de x durante a avaliação
        'sin': np.sin,
        'cos': np.cos,
        'tan': np.tan,
        'exp': np.exp,
        'log': np.log,
        'sqrt': np.sqrt,
        'pi': np.pi,
        'e': np.e,
        'abs': np.abs
    }
    
    # A função real que será usada nos cálculos
    def f(x_val):
        # Ignora avisos do numpy (como divisão por zero, que trataremos depois)
        with warnings.catch_warnings():
            warnings.simplefilter("ignore")
            # Atualiza o valor de x no dicionário de operações
            operacoes_permitidas['x'] = x_val
            try:
                # O eval avalia a expressão matemática usando apenas as operações permitidas
                resultado = eval(expressao, {"__builtins__": {}}, operacoes_permitidas)
                return float(resultado)
            except Exception as e:
                # Se algo der errado (ex: divisão por zero), retorna "Not a Number" (NaN)
                return np.nan
                
    return f

def verificar_valores_reais(f, a, b, num_pontos=100):
    """
    Verifica se a função retorna valores reais (não complexos) no intervalo [a, b].
    Fazemos isso testando a função em vários pontos do intervalo.
    """
    # Cria uma lista de 'num_pontos' igualmente espaçados entre 'a' e 'b'
    pontos = np.linspace(a, b, num_pontos)
    
    for x in pontos:
        valor = f(x)
        # Verifica se o valor é um número complexo (ex: raiz de número negativo)
        if isinstance(valor, complex):
            return False, f"A função retorna valor complexo em x = {x:.4f}"
        # Verifica se o valor é indefinido (NaN) ou infinito (ex: divisão por zero)
        if np.isnan(valor) or np.isinf(valor):
            return False, f"A função é indefinida ou infinita em x = {x:.4f}"
        # CORRIGIDO: Removemos o limite arbitrário de 1e10 para não rejeitar funções legítimas com valores grandes
            
    return True, "A função retorna apenas valores reais."

def verificar_continuidade(f, a, b, num_pontos=1000):
    """
    Verifica se a função é contínua no intervalo [a, b].
    Fazemos isso analisando se há 'saltos' muito grandes entre pontos muito próximos.
    
    NOTA IMPORTANTE: Esta é uma verificação heurística, não rigorosa. Uma verificação
    matemática rigorosa de continuidade exigiria análise simbólica ou teoremas específicos.
    Este método detecta descontinuidades óbvias mas pode deixar passar descontinuidades sutis.
    """
    # Cria muitos pontos próximos entre 'a' e 'b'
    pontos = np.linspace(a, b, num_pontos)
    valores = [f(x) for x in pontos]
    
    # Calcula o comprimento do intervalo para normalizar o limite de detecção
    comprimento_intervalo = b - a
    # CORRIGIDO: O limite agora é proporcional ao intervalo, não arbitrário
    # Usa 10% da amplitude máxima de valores como limite
    amplitude = max(valores) - min(valores)
    if amplitude == 0:
        amplitude = 1  # Evita divisão por zero para funções constantes
    limite_deteccao = amplitude * 0.1
    
    # Verifica a diferença entre cada ponto e o próximo
    for i in range(len(valores) - 1):
        # Se a diferença entre dois pontos muito próximos for muito grande,
        # provavelmente há um "salto" (descontinuidade)
        if abs(valores[i+1] - valores[i]) > limite_deteccao:
            return False, f"Possível descontinuidade detectada perto de x = {pontos[i]:.4f}"
            
    return True, "A função parece ser contínua no intervalo."

def metodo_bissecao(f_str, a, b, tolerancia, max_iter=100):
    """
    Função principal que executa o Método da Bisseção.
    Este método se baseia no Teorema do Anulamento (e no Teorema do Valor Intermediário),
    conforme abordado no Capítulo 2 e 3 do seu TCC.
    """
    print(f"\n--- INICIANDO ANÁLISE PARA f(x) = {f_str} NO INTERVALO [{a}, {b}] ---\n")
    
    # 1. Converte a string do usuário em uma função de verdade
    f = criar_funcao_segura(f_str)
    
    # 2. Validação: A função retorna apenas números reais e é definida no intervalo?
    print("Verificando se a função retorna valores reais e definidos...")
    ok_reais, msg_reais = verificar_valores_reais(f, a, b)
    if not ok_reais:
        print(f"ERRO: {msg_reais}")
        print("O Método da Bisseção exige que a função seja definida e real em todo o intervalo.")
        return None
    print("✓ OK!\n")
    
    # 3. Validação: A função é contínua?
    # (Condição essencial para o Teorema do Anulamento funcionar)
    print("Verificando a continuidade da função no intervalo...")
    ok_cont, msg_cont = verificar_continuidade(f, a, b)
    if not ok_cont:
        print(f"ERRO: {msg_cont}")
        print("O Método da Bisseção (baseado no Teorema do Anulamento) exige uma função contínua.")
        return None
    print("✓ OK!\n")
    
    # 4. Validação: Condição de sinais opostos (Teorema do Anulamento)
    # Se f(a) e f(b) têm sinais opostos, então f(a) * f(b) < 0
    fa = f(a)
    fb = f(b)
    
    print(f"Verificando a condição de sinais opostos: f({a}) = {fa:.4f} e f({b}) = {fb:.4f}")
    
    # CORRIGIDO: Agora tratamos o caso onde a raiz está exatamente em um dos extremos
    # Se f(a) == 0, a raiz está em 'a'
    if abs(fa) < 1e-10:  # Comparação com tolerância, não == 0
        print(f"✓ Raiz encontrada exatamente em a = {a}")
        print("\n--- RESULTADO FINAL ---")
        print(f"Raiz encontrada: {a:.6f}")
        print(f"Número de iterações: 0")
        print(f"Erro relativo final: 0.0")
        print(f"Valor da função na raiz: f({a:.6f}) = {fa:.6e}")
        return a
    
    # Se f(b) == 0, a raiz está em 'b'
    if abs(fb) < 1e-10:  # Comparação com tolerância, não == 0
        print(f"✓ Raiz encontrada exatamente em b = {b}")
        print("\n--- RESULTADO FINAL ---")
        print(f"Raiz encontrada: {b:.6f}")
        print(f"Número de iterações: 0")
        print(f"Erro relativo final: 0.0")
        print(f"Valor da função na raiz: f({b:.6f}) = {fb:.6e}")
        return b
    
    # Agora verifica se há mudança de sinal
    if fa * fb >= 0:
        print("ERRO: f(a) e f(b) não possuem sinais opostos!")
        print("Isso significa que o Teorema do Anulamento não garante uma raiz neste intervalo.")
        print("Por favor, escolha outro intervalo onde um valor seja positivo e o outro negativo.")
        return None
    print("✓ OK! A condição f(a) * f(b) < 0 foi satisfeita.\n")
    
    # --- INÍCIO DO MÉTODO DA BISSEÇÃO ---
    print("--- INICIANDO AS ITERAÇÕES DO MÉTODO DA BISSEÇÃO ---")
    print("Iter |       a      |       b      |      xM      |     f(xM)    | Erro Relativo")
    print("-" * 80)
    
    iteracao = 1
    erro_relativo = None  # CORRIGIDO: Começa como None em vez de 1.0 para indicar indefinido
    xM_anterior = None    # CORRIGIDO: Começa como None
    
    while iteracao <= max_iter:
        # Passo 1: Calcula o ponto médio (xM)
        xM = (a + b) / 2.0
        fxM = f(xM)
        
        # Passo 2: Calcula o erro relativo (a partir da segunda iteração)
        # CORRIGIDO: Agora tratamos corretamente o erro relativo
        if iteracao > 1 and xM_anterior is not None:
            # Erro relativo = |(x_atual - x_anterior) / x_atual|
            # Usamos abs() porque o erro é sempre positivo
            if abs(xM) > 1e-15:  # Comparação com tolerância para evitar divisão por zero
                erro_relativo = abs((xM - xM_anterior) / xM)
            else:
                # Se xM é muito próximo de zero, usa erro absoluto
                erro_relativo = abs(xM - xM_anterior)
        else:
            # Na primeira iteração, não há erro relativo definido
            erro_relativo = None
        
        # Imprime a linha da tabela com os dados desta iteração
        # CORRIGIDO: Mostra "N/A" em vez de 1.0 para a primeira iteração
        if erro_relativo is None:
            print(f"{iteracao:4d} | {a:12.6f} | {b:12.6f} | {xM:12.6f} | {fxM:12.6f} |      N/A")
        else:
            print(f"{iteracao:4d} | {a:12.6f} | {b:12.6f} | {xM:12.6f} | {fxM:12.6f} | {erro_relativo:12.6f}")
        
        # Passo 3: Critério de Parada
        # Se encontramos a raiz exata ou o erro é menor que a tolerância, paramos!
        # CORRIGIDO: Comparação com tolerância em vez de == 0
        if abs(fxM) < 1e-10 or (iteracao > 1 and erro_relativo is not None and erro_relativo < tolerancia):
            print("-" * 80)
            print("\n--- RESULTADO FINAL ---")
            print(f"Raiz encontrada: {xM:.6f}")
            print(f"Número de iterações: {iteracao}")
            if erro_relativo is not None:
                print(f"Erro relativo final: {erro_relativo:.6e}")
            else:
                print(f"Erro relativo final: N/A (raiz exata encontrada)")
            print(f"Valor da função na raiz: f({xM:.6f}) = {fxM:.6e}")
            return xM
            
        # Passo 4: Atualiza o intervalo [a, b] de acordo com a regra de sinais
        # Se f(a) e f(xM) têm sinais opostos, a raiz está na metade esquerda
        if f(a) * fxM < 0:
            b = xM
        # Senão, a raiz está na metade direita
        else:
            a = xM
            
        xM_anterior = xM
        iteracao += 1
        
    # Se o loop terminar porque atingiu o limite de iterações
    print("\nAVISO: O limite máximo de iterações foi atingido!")
    print(f"Última estimativa da raiz: {xM:.6f}")
    return xM

def executar_interativo():
    """
    Função que interage com o usuário, pedindo os dados de entrada.
    """
    print("=" * 60)
    print("   PROGRAMA DO MÉTODO DA BISSEÇÃO - APLICAÇÃO PRÁTICA (TCC)")
    print("=" * 60)
    print("\nEste programa encontra a raiz de uma função usando o Método da Bisseção.")
    print("Exemplos de funções que você pode digitar:")
    print(" - x**2 - 4         (significa x ao quadrado menos 4)")
    print(" - x**3 + 2*x - 1   (significa x ao cubo mais 2x menos 1)")
    print(" - sin(x) - 0.5     (seno de x menos 0.5)")
    
    try:
        f_str = input("\n1. Digite a função f(x): ")
        a = float(input("2. Digite o início do intervalo (ponto 'a'): "))
        b = float(input("3. Digite o fim do intervalo (ponto 'b'): "))
        tolerancia = float(input("4. Digite a tolerância de erro (ex: 0.0001): "))
        
        if a >= b:
            print("\nERRO: O ponto 'a' deve ser menor que o ponto 'b'.")
            return
        
        if tolerancia <= 0:
            print("\nERRO: A tolerância deve ser um número positivo.")
            return
            
        metodo_bissecao(f_str, a, b, tolerancia)
        
    except ValueError:
        print("\nERRO: Você digitou um valor inválido. Certifique-se de digitar números para 'a', 'b' e tolerância.")
    except Exception as e:
        print(f"\nERRO INESPERADO: {e}")

if __name__ == "__main__":
    # Se o script for executado diretamente, roda o modo interativo
    executar_interativo()
