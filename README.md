# Mercado-Financeiro
Ferramenta didática em Python sobre Bandas de Bollinger no mercado financeiro brasileiro.
# Instalação das bibliotecas necessárias
!pip install yfinance pandas plotly -q

import yfinance as yf
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import warnings
warnings.filterwarnings('ignore')
import plotly.graph_objects as go


def calcular_bandas_bollinger(dados, periodo=20, desvios=2):
    dados['SMA'] = dados['Close'].rolling(window=periodo).mean()
    dados['STD'] = dados['Close'].rolling(window=periodo).std()
    dados['Banda_Superior'] = dados['SMA'] + (desvios * dados['STD'])
    dados['Banda_Inferior'] = dados['SMA'] - (desvios * dados['STD'])
    return dados


def verificar_sinais(dados):
    ultimo_preco = dados['Close'].iloc[-1]
    banda_superior = dados['Banda_Superior'].iloc[-1]
    banda_inferior = dados['Banda_Inferior'].iloc[-1]
    sma = dados['SMA'].iloc[-1]

    sinais = []

    if ultimo_preco >= banda_superior:
        sinais.append('🔴 SOBRECOMPRA - Preço acima da banda superior!')
    elif ultimo_preco <= banda_inferior:
        sinais.append('🟢 SOBREVENDA - Preço abaixo da banda inferior!')
    elif ultimo_preco > sma and (ultimo_preco / banda_superior) > 0.98:
        sinais.append('⚠ ATENÇÃO - Preço se aproximando da banda superior')
    elif ultimo_preco < sma and (ultimo_preco / banda_inferior) < 1.02:
        sinais.append('⚠ ATENÇÃO - Preço se aproximando da banda inferior')
    else:
        sinais.append('✅ Preço dentro das bandas normais')

    return sinais


def plotar_grafico(dados, ticker):
    fig = go.Figure()

    fig.add_trace(go.Scatter(
        x=dados.index, y=dados['Close'], mode='lines',
        name='Preço',
        hovertemplate='Data: %{x|%d/%m/%Y}<br>Preço: R$ %{y:.2f}<extra></extra>'
    ))

    fig.add_trace(go.Scatter(
        x=dados.index, y=dados['SMA'], mode='lines', name='Média Móvel (20)'
    ))

    fig.add_trace(go.Scatter(
        x=dados.index, y=dados['Banda_Superior'], mode='lines',
        name='Banda Superior', line=dict(dash='dash')
    ))

    fig.add_trace(go.Scatter(
        x=dados.index, y=dados['Banda_Inferior'], mode='lines',
        name='Banda Inferior', line=dict(dash='dash')
    ))

    fig.update_layout(
        title=f'{ticker} - Bandas de Bollinger',
        xaxis_title='Data',
        yaxis_title='Preço (R$)',
        hovermode='x unified'
    )

    fig.show()


def obter_dados_ativo(ticker, periodo='3mo'):
    if not ticker.endswith('.SA'):
        ticker = ticker + '.SA'

    print(f"📊 Buscando dados para {ticker}...")

    try:
        ativo = yf.Ticker(ticker)
        dados = ativo.history(period=periodo)

        if dados.empty:
            print(f"❌ Erro: Não foi possível obter dados para {ticker}")
            return None

        return dados, ticker

    except Exception as e:
        print(f"❌ Erro ao buscar dados: {e}")
        return None


def mostrar_informacoes(dados, ticker, sinais):
    ultimo_preco = dados['Close'].iloc[-1]
    variacao = ((dados['Close'].iloc[-1] / dados['Close'].iloc[-2]) - 1) * 100
    banda_sup = dados['Banda_Superior'].iloc[-1]
    banda_inf = dados['Banda_Inferior'].iloc[-1]
    sma = dados['SMA'].iloc[-1]

    print("\n" + "="*60)
    print(f"📈 INFORMAÇÕES DO ATIVO: {ticker}")
    print("="*60)
    print(f"💰 Preço Atual: R$ {ultimo_preco:.2f}")
    print(f"📊 Variação: {variacao:+.2f}%")
    print(f"📅 Última Atualização: {dados.index[-1].strftime('%d/%m/%Y %H:%M')}")
    print("\n" + "-"*60)
    print("📊 BANDAS DE BOLLINGER:")
    print("-"*60)
    print(f"🔴 Banda Superior: R$ {banda_sup:.2f}")
    print(f"🟡 Média Móvel (20): R$ {sma:.2f}")
    print(f"🟢 Banda Inferior: R$ {banda_inf:.2f}")
    print("\n" + "-"*60)
    print("🚨 ANÁLISE:")
    print("-"*60)
    for sinal in sinais:
        print(f"   {sinal}")
    print("="*60 + "\n")


print("="*60)
print("🇧🇷 MONITOR DE ATIVOS B3 COM BANDAS DE BOLLINGER")
print("="*60)
print("\nExemplos de ativos:")
print("  • PETR4 (Petrobras)")
print("  • VALE3 (Vale)")
print("  • ITUB4 (Itaú)")
print("  • BBAS3 (Banco do Brasil)")
print("  • MGLU3 (Magazine Luiza)")
print("  • WEGE3 (Weg)")

ticker_input = input("\n💼 Digite o código do ativo (ex: PETR4): ").strip().upper()

resultado = obter_dados_ativo(ticker_input, periodo='3mo')

if resultado:
    dados, ticker_completo = resultado
    dados = calcular_bandas_bollinger(dados)
    sinais = verificar_sinais(dados)
    mostrar_informacoes(dados, ticker_completo, sinais)
    plotar_grafico(dados, ticker_completo)
    print("✅ Análise concluída com sucesso!")

else:
    print("\n❌ Não foi possível realizar a análise. Verifique o código do ativo.")
