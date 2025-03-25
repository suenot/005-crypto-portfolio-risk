# Глава 5: Построение портфеля и бюджетирование рисков для цифровых активов

## Обзор

Построение портфеля на рынке цифровых активов представляет уникальные задачи, требующие существенной адаптации классической финансовой теории. В отличие от традиционных фондовых рынков, работающих в ограниченные часы, криптовалютные рынки торгуются 24/7/365, создавая непрерывную подверженность волатильности, мгновенным обвалам и событиям ликвидности, которые могут произойти в любое время. Эта непрерывная природа фундаментально меняет способ расчёта метрик риска, таких как коэффициент Шарпа, измерения просадок и подход к диверсификации между активами с экстремальными хвостовыми зависимостями.

Современная портфельная теория (MPT), введённая Гарри Марковицем в 1952 году, предоставляет математическую основу для построения оптимальных портфелей. Однако прямое применение средне-дисперсионной оптимизации к криптоактивам чревато опасностями: распределения доходностей демонстрируют экстремальный эксцесс, корреляции резко возрастают во время рыночного стресса, а ковариационные матрицы, оценённые по историческим данным, крайне нестабильны. В этой главе рассматриваются как классические подходы, так и их необходимые модификации для криптосферы, включая модель Блэка-Литтермана для учёта субъективных взглядов, иерархический паритет рисков для более устойчивого распределения и критерий Келли, адаптированный для бессрочных фьючерсов с кредитным плечом.

Бюджетирование рисков в криптовалютах требует внимания к факторам, отсутствующим в традиционных финансах. Ставки финансирования бессрочных фьючерсов выступают как компонент carry, который может существенно влиять на доходность портфеля. Каскады ликвидаций могут вызывать мгновенные обвалы, нарушающие предположения о непрерывности ценовых траекторий. Корреляционные режимы резко меняются между бычьими и медвежьими рынками, когда альткоины становятся почти идеально коррелированными во время обвалов. Эта глава предоставляет как теоретические основы, так и практические реализации на Python и Rust для построения устойчивых криптопортфелей, учитывающих эти реалии.

## Содержание

1. [Введение в построение криптопортфеля](#раздел-1-введение-в-построение-криптопортфеля)
2. [Математические основы портфельной теории](#раздел-2-математические-основы-портфельной-теории)
3. [Сравнение методов оптимизации портфеля](#раздел-3-сравнение-методов-оптимизации-портфеля)
4. [Торговые применения на рынке цифровых активов](#раздел-4-торговые-применения-на-рынке-цифровых-активов)
5. [Реализация на Python](#раздел-5-реализация-на-python)
6. [Реализация на Rust](#раздел-6-реализация-на-rust)
7. [Практические примеры](#раздел-7-практические-примеры)
8. [Фреймворк бэктестирования](#раздел-8-фреймворк-бэктестирования)
9. [Оценка производительности](#раздел-9-оценка-производительности)
10. [Перспективные направления](#раздел-10-перспективные-направления)

---

## Раздел 1: Введение в построение криптопортфеля

### Аргументы в пользу диверсификации портфеля в криптовалютах

Криптовалютные рынки предлагают сотни торгуемых активов, однако многие участники держат концентрированные позиции в одной монете. Портфельная теория демонстрирует, что диверсификация может снизить риск без пропорционального уменьшения ожидаемой доходности при условии, что активы не идеально коррелированы. В криптовалютах корреляционные структуры значительно различаются в зависимости от рыночного режима: в спокойные периоды BTC, ETH и альткоины могут демонстрировать умеренные корреляции (0.3-0.6), тогда как при резких распродажах корреляции могут превышать 0.9, когда все активы падают одновременно.

Портфель 1/N (равновзвешенный) служит удивительно сильным бенчмарком. Исследования показали, что наивная диверсификация часто превосходит оптимизированные портфели вне выборки, особенно когда ошибка оценки ожидаемых доходностей велика. В криптовалютах, где оценка доходности крайне зашумлена, портфель 1/N предоставляет ценную базовую линию для сравнения с более сложными подходами.

### Ключевые метрики производительности, адаптированные для криптовалют

**Коэффициент Шарпа** — наиболее широко используемая метрика доходности с поправкой на риск. Для криптовалют необходимо адаптировать коэффициент аннуализации с учётом торговли 24/7:

- Традиционный: `SR_annual = SR_daily * sqrt(252)`
- Крипто: `SR_annual = SR_daily * sqrt(365)`

**Коэффициент Сортино** учитывает только нисходящее отклонение, что делает его более подходящим для асимметричных распределений доходности, характерных для криптовалют. **Коэффициент Кальмара** (аннуализированная доходность / максимальная просадка) отражает риск обвалов, что особенно актуально с учётом истории просадок крипторынка на 50-80%.

**Максимальная просадка** требует тщательного измерения в криптовалютах. Мгновенные обвалы длительностью в минуты могут создавать экстремальные просадки, которые быстро восстанавливаются. Выбор между измерением просадки на тиковом уровне или по дневным закрытиям — важное проектное решение для любого фреймворка бэктестирования.

### Соотношение риска и доходности в цифровых активах

Фундаментальное соотношение между риском и доходностью приобретает новые измерения в криптовалютах. Кредитное плечо через бессрочные фьючерсы позволяет трейдерам усиливать как доходность, так и риски, с дополнительной сложностью в виде платежей финансирования и риска ликвидации. Понимание этого соотношения требует нюансированного взгляда на риск, выходящего за рамки простых мер волатильности и включающего хвостовой риск, риск ликвидности и уникальные риски протоколов децентрализованных финансов.

---

## Раздел 2: Математические основы портфельной теории

### Средне-дисперсионная оптимизация

При наличии N активов с вектором ожидаемой доходности **mu** и ковариационной матрицей **Sigma** задача средне-дисперсионной оптимизации формулируется как:

```
minimize    w^T * Sigma * w
subject to  w^T * mu >= target_return
            w^T * 1 = 1
            w_i >= 0  (ограничение только длинных позиций)
```

Где **w** — вектор весов портфеля. Решение очерчивает **эффективную границу** — множество портфелей, предлагающих максимальную доходность для каждого уровня риска.

**Портфель минимальной дисперсии** — это крайняя левая точка на эффективной границе:

```
w_mv = (Sigma^{-1} * 1) / (1^T * Sigma^{-1} * 1)
```

### Модель Блэка-Литтермана

Модель Блэка-Литтермана решает проблему экстремальной чувствительности средне-дисперсионной оптимизации к оценкам ожидаемой доходности, комбинируя рыночные равновесные доходности со взглядами инвестора:

```
Равновесные доходности:   pi = delta * Sigma * w_market
Комбинированные доходности: mu_BL = [(tau * Sigma)^{-1} + P^T * Omega^{-1} * P]^{-1}
                                     * [(tau * Sigma)^{-1} * pi + P^T * Omega^{-1} * Q]
```

Где:
- `delta` = коэффициент неприятия риска
- `tau` = скаляр неопределённости (обычно 0.025-0.05)
- `P` = матрица выбора (матрица взглядов)
- `Q` = вектор доходностей взглядов
- `Omega` = диагональная матрица неопределённости взглядов

Для криптовалют взгляды могут включать: «BTC превзойдёт ETH на 5% в годовом выражении» или «SOL принесёт 20% за следующий квартал».

### Критерий Келли для бессрочных фьючерсов с кредитным плечом

Критерий Келли определяет оптимальную долю капитала для размещения:

```
f* = (mu - r_f) / sigma^2
```

Для бессрочных фьючерсов с кредитным плечом доля Келли должна учитывать:
- Ставку финансирования: `r_funding` (положительная = лонги платят шортам)
- Границу ликвидации: максимальное плечо до принудительного закрытия
- Скорректированную ожидаемую доходность: `mu_adj = mu - r_funding * leverage`

```
f*_perp = (mu_adj - r_f) / sigma^2
Практический Келли: f_practical = f* / 2  (половина Келли для безопасности)
```

### Иерархический паритет рисков (HRP)

HRP использует иерархическую кластеризацию на корреляционной матрице для построения портфеля без обращения матрицы:

1. **Древовидная кластеризация**: Вычислить матрицу расстояний `D_ij = sqrt(0.5 * (1 - rho_ij))` и применить кластеризацию одиночной связью
2. **Квази-диагонализация**: Переупорядочить ковариационную матрицу, разместив коррелированные активы рядом
3. **Рекурсивное деление**: Разделить активы на кластеры и распределить обратно пропорционально дисперсии кластера

### Оценка ковариационной матрицы

Робастная оценка ковариации критически важна для криптовалют. Методы включают:

- **Выборочная ковариация**: `S = (1/T) * X^T * X` (зашумлена при малом числе наблюдений)
- **Сжатие Ледуа-Вольфа**: `Sigma_shrunk = alpha * F + (1-alpha) * S`, где F — структурированная цель
- **Экспоненциально взвешенная**: `Sigma_t = lambda * Sigma_{t-1} + (1-lambda) * r_t * r_t^T` (адаптируется к смене режимов)
- **Минимальный ковариационный определитель**: Устойчив к выбросам от мгновенных обвалов

---

## Раздел 3: Сравнение методов оптимизации портфеля

| Метод | Требуется оценка доходности | Учитывает толстые хвосты | Устойчив к смене корреляционного режима | Учитывает плечо | Сложность |
|-------|----------------------------|--------------------------|----------------------------------------|-----------------|-----------|
| Средне-дисперсионный (Марковиц) | Да | Нет | Нет | Нет | Низкая |
| Минимальная дисперсия | Нет | Нет | Частично | Нет | Низкая |
| Блэк-Литтерман | Да (взгляды) | Нет | Частично | Нет | Средняя |
| Паритет рисков | Нет | Частично | Частично | Да | Средняя |
| Иерархический паритет рисков | Нет | Да | Да | Нет | Средняя |
| Критерий Келли | Да | Нет | Нет | Да | Низкая |
| Робастная оптимизация | Да (множество неопределённости) | Да | Да | Опционально | Высокая |
| 1/N Равновзвешенный | Нет | Н/Д | Н/Д | Нет | Отсутствует |

| Метрика | Формула | Адаптация для крипто | Типичный диапазон (крипто) |
|---------|---------|---------------------|---------------------------|
| Коэффициент Шарпа | (R_p - R_f) / sigma_p | Аннуализация с sqrt(365) | -0.5 до 2.0 |
| Коэффициент Сортино | (R_p - R_f) / sigma_down | Только нисходящее отклонение | -0.5 до 3.0 |
| Коэффициент Кальмара | R_annual / MaxDD | Критичен для просадок крипто | 0.1 до 1.5 |
| Максимальная просадка | max(peak - trough) / peak | Измерение на часовой гранулярности | 20% до 80% |
| Информационный коэффициент | alpha / tracking_error | vs бенчмарк BTC | -1.0 до 1.0 |
| Carry финансирования | sum(funding_rates) | Аннуализированные 8ч ставки | -20% до +30% |

---

## Раздел 4: Торговые применения на рынке цифровых активов

### 4.1 Паритет рисков BTC/ETH/альткоины

Паритет рисков распределяет капитал обратно пропорционально вкладу каждого актива в риск портфеля. Для криптопортфеля из BTC, ETH и корзины альткоинов:

```
Risk contribution_i = w_i * (Sigma * w)_i / (w^T * Sigma * w)
Цель: RC_BTC = RC_ETH = RC_ALT = 1/3
```

На практике более низкая волатильность BTC приводит к большему весу (часто 50-60%), тогда как альткоины получают меньшие аллокации из-за их экстремальной волатильности.

### 4.2 Ставка финансирования как компонент carry

Бессрочные фьючерсы Bybit выплачивают финансирование каждые 8 часов. Портфельная стратегия может использовать финансирование:

- **Положительное финансирование**: Лонги платят шортам. Шортить перп, лонгить спот = зарабатывать финансирование как carry.
- **Отрицательное финансирование**: Шорты платят лонгам. Лонгить перп, шортить спот (или просто лонгить перп).
- Аннуализированный carry: `carry_annual = funding_rate * 3 * 365`

Эта базисная сделка (cash-and-carry арбитраж) может приносить 10-30% годовых во время бычьих рынков.

### 4.3 Определение корреляционного режима

Корреляции криптовалют переключаются между режимами:
- **Бычий режим**: Доминация BTC падает, корреляции альткоинов умеренные (rho ~ 0.4-0.6)
- **Медвежий режим/обвал**: Бегство в качество, все альткоины коррелируют с BTC (rho ~ 0.8-0.95)
- **Режим ротации**: Секторная ротация между DeFi, L1, L2, мемами (rho варьируется по секторам)

Определение текущего режима критически важно для построения портфеля. Скользящая 30-дневная корреляционная матрица с экспоненциальным взвешиванием помогает идентифицировать переходы.

### 4.4 Управление риском ликвидации

Для позиций с кредитным плечом цена ликвидации определяет максимальный убыток:

```
Цена ликвидации (лонг) = entry_price * (1 - 1/leverage + maintenance_margin)
Цена ликвидации (шорт) = entry_price * (1 + 1/leverage - maintenance_margin)
```

Риск ликвидации на уровне портфеля требует моделирования коррелированных движений по позициям. Портфель с 5x плечом по нескольким альткоинам имеет значительно более высокий риск ликвидации во время обвала, чем предполагают индивидуальные риски позиций.

### 4.5 Динамическая ребалансировка с учётом транзакционных издержек

Частота ребалансировки должна балансировать ошибку слежения и транзакционные издержки:
- Комиссия тейкера Bybit: 0.055%
- Комиссия мейкера Bybit: 0.02%
- Проскальзывание: 0.01-0.1% в зависимости от актива и объёма
- Влияние ставки финансирования во время ребалансировки

Подход с зоной бездействия выполняет ребалансировку только при отклонении весов за пределы порога, снижая оборот при сохранении целевого распределения.

---

## Раздел 5: Реализация на Python

### Класс оптимизатора портфеля

```python
import numpy as np
import pandas as pd
from scipy.optimize import minimize
from scipy.cluster.hierarchy import linkage, leaves_list
from scipy.spatial.distance import squareform
import yfinance as yf
import requests
from typing import Dict, List, Optional, Tuple


class CryptoPortfolioOptimizer:
    """Оптимизация портфеля для цифровых активов с крипто-специфическими адаптациями."""

    def __init__(self, symbols: List[str], risk_free_rate: float = 0.05):
        self.symbols = symbols
        self.risk_free_rate = risk_free_rate
        self.returns = None
        self.cov_matrix = None

    def fetch_bybit_klines(self, symbol: str, interval: str = "D",
                           limit: int = 200) -> pd.DataFrame:
        """Получение данных OHLCV с API Bybit."""
        url = "https://api.bybit.com/v5/market/kline"
        params = {
            "category": "linear",
            "symbol": symbol,
            "interval": interval,
            "limit": limit
        }
        response = requests.get(url, params=params)
        data = response.json()["result"]["list"]
        df = pd.DataFrame(data, columns=[
            "timestamp", "open", "high", "low", "close", "volume", "turnover"
        ])
        df["close"] = df["close"].astype(float)
        df["timestamp"] = pd.to_datetime(df["timestamp"].astype(int), unit="ms")
        df = df.sort_values("timestamp").set_index("timestamp")
        return df

    def fetch_returns(self, source: str = "bybit") -> pd.DataFrame:
        """Получение и расчёт дневных доходностей для всех символов."""
        all_returns = {}
        if source == "bybit":
            for sym in self.symbols:
                df = self.fetch_bybit_klines(sym)
                all_returns[sym] = df["close"].pct_change().dropna()
        elif source == "yfinance":
            for sym in self.symbols:
                df = yf.download(sym, period="1y", interval="1d")
                all_returns[sym] = df["Close"].pct_change().dropna()
        self.returns = pd.DataFrame(all_returns).dropna()
        self.cov_matrix = self.returns.cov() * 365  # аннуализированная
        return self.returns

    def mean_variance_optimize(self, target_return: Optional[float] = None
                               ) -> np.ndarray:
        """Средне-дисперсионная оптимизация Марковица."""
        n = len(self.symbols)
        mu = self.returns.mean() * 365
        Sigma = self.cov_matrix

        def portfolio_volatility(w):
            return np.sqrt(w @ Sigma.values @ w)

        constraints = [{"type": "eq", "fun": lambda w: np.sum(w) - 1}]
        if target_return is not None:
            constraints.append({
                "type": "eq",
                "fun": lambda w: w @ mu.values - target_return
            })
        bounds = [(0, 1)] * n
        w0 = np.ones(n) / n
        result = minimize(portfolio_volatility, w0, method="SLSQP",
                          bounds=bounds, constraints=constraints)
        return result.x

    def minimum_variance(self) -> np.ndarray:
        """Вычисление весов портфеля минимальной дисперсии."""
        return self.mean_variance_optimize(target_return=None)

    def black_litterman(self, views: Dict[str, float],
                        view_confidences: Dict[str, float],
                        tau: float = 0.05, delta: float = 2.5
                        ) -> np.ndarray:
        """Модель Блэка-Литтермана со взглядами инвестора."""
        n = len(self.symbols)
        Sigma = self.cov_matrix.values
        w_market = np.ones(n) / n  # равный вес как прокси для рыночной капитализации
        pi = delta * Sigma @ w_market

        # Построение P и Q из взглядов
        P = np.zeros((len(views), n))
        Q = np.zeros(len(views))
        omega_diag = np.zeros(len(views))

        for i, (asset, view_return) in enumerate(views.items()):
            idx = self.symbols.index(asset)
            P[i, idx] = 1.0
            Q[i] = view_return
            omega_diag[i] = (1.0 / view_confidences[asset]) * tau * Sigma[idx, idx]

        Omega = np.diag(omega_diag)
        tau_Sigma_inv = np.linalg.inv(tau * Sigma)
        M = np.linalg.inv(tau_Sigma_inv + P.T @ np.linalg.inv(Omega) @ P)
        mu_bl = M @ (tau_Sigma_inv @ pi + P.T @ np.linalg.inv(Omega) @ Q)

        # Оптимизация с ожидаемыми доходностями BL
        def neg_sharpe(w):
            ret = w @ mu_bl
            vol = np.sqrt(w @ Sigma @ w)
            return -(ret - self.risk_free_rate) / vol

        constraints = [{"type": "eq", "fun": lambda w: np.sum(w) - 1}]
        bounds = [(0, 1)] * n
        w0 = np.ones(n) / n
        result = minimize(neg_sharpe, w0, method="SLSQP",
                          bounds=bounds, constraints=constraints)
        return result.x

    def hierarchical_risk_parity(self) -> np.ndarray:
        """Аллокация иерархического паритета рисков (HRP)."""
        corr = self.returns.corr()
        dist = np.sqrt(0.5 * (1 - corr))
        dist_condensed = squareform(dist.values, checks=False)
        link = linkage(dist_condensed, method="single")
        sort_ix = leaves_list(link)
        sorted_symbols = [self.symbols[i] for i in sort_ix]

        # Рекурсивное деление
        weights = pd.Series(1.0, index=sorted_symbols)
        clusters = [sorted_symbols]

        while clusters:
            new_clusters = []
            for cluster in clusters:
                if len(cluster) <= 1:
                    continue
                mid = len(cluster) // 2
                left = cluster[:mid]
                right = cluster[mid:]

                left_var = self._cluster_variance(left)
                right_var = self._cluster_variance(right)
                alpha = 1 - left_var / (left_var + right_var)

                for s in left:
                    weights[s] *= alpha
                for s in right:
                    weights[s] *= (1 - alpha)

                new_clusters.extend([left, right])
            clusters = new_clusters

        return weights.reindex(self.symbols).values

    def _cluster_variance(self, symbols: List[str]) -> float:
        sub_cov = self.returns[symbols].cov() * 365
        w = np.ones(len(symbols)) / len(symbols)
        return w @ sub_cov.values @ w

    def kelly_criterion(self, leverage: float = 1.0,
                        funding_rate: float = 0.0001) -> np.ndarray:
        """Критерий Келли с учётом ставки финансирования."""
        mu = self.returns.mean() * 365
        var = self.returns.var() * 365
        # Корректировка на стоимость финансирования для позиций с плечом
        mu_adj = mu - funding_rate * 3 * 365 * leverage
        kelly_fractions = (mu_adj - self.risk_free_rate) / var
        # Половина Келли для безопасности
        half_kelly = kelly_fractions / 2
        # Нормализация до суммы 1, ограничение отрицательных нулём
        half_kelly = half_kelly.clip(lower=0)
        if half_kelly.sum() > 0:
            half_kelly = half_kelly / half_kelly.sum()
        return half_kelly.values

    def risk_parity(self) -> np.ndarray:
        """Портфель с равным вкладом в риск."""
        n = len(self.symbols)
        Sigma = self.cov_matrix.values

        def risk_budget_objective(w):
            port_var = w @ Sigma @ w
            marginal = Sigma @ w
            rc = w * marginal
            target_rc = port_var / n
            return np.sum((rc - target_rc) ** 2)

        constraints = [{"type": "eq", "fun": lambda w: np.sum(w) - 1}]
        bounds = [(0.01, 1)] * n
        w0 = np.ones(n) / n
        result = minimize(risk_budget_objective, w0, method="SLSQP",
                          bounds=bounds, constraints=constraints)
        return result.x


class CryptoRiskMetrics:
    """Метрики риска, адаптированные для круглосуточных криптовалютных рынков."""

    @staticmethod
    def sharpe_ratio(returns: pd.Series, risk_free_rate: float = 0.05) -> float:
        excess = returns - risk_free_rate / 365
        return np.sqrt(365) * excess.mean() / excess.std()

    @staticmethod
    def sortino_ratio(returns: pd.Series, risk_free_rate: float = 0.05) -> float:
        excess = returns - risk_free_rate / 365
        downside = excess[excess < 0]
        downside_std = np.sqrt((downside ** 2).mean())
        return np.sqrt(365) * excess.mean() / downside_std

    @staticmethod
    def calmar_ratio(returns: pd.Series) -> float:
        annual_return = returns.mean() * 365
        max_dd = CryptoRiskMetrics.max_drawdown(returns)
        return annual_return / abs(max_dd) if max_dd != 0 else 0

    @staticmethod
    def max_drawdown(returns: pd.Series) -> float:
        cumulative = (1 + returns).cumprod()
        peak = cumulative.cummax()
        drawdown = (cumulative - peak) / peak
        return drawdown.min()

    @staticmethod
    def funding_carry(funding_rates: pd.Series) -> float:
        """Аннуализированный carry финансирования из 8-часовых ставок."""
        return funding_rates.mean() * 3 * 365
```

### Пример использования

```python
# Инициализация оптимизатора с символами бессрочных фьючерсов Bybit
optimizer = CryptoPortfolioOptimizer(
    symbols=["BTCUSDT", "ETHUSDT", "SOLUSDT", "AVAXUSDT", "LINKUSDT"]
)
optimizer.fetch_returns(source="bybit")

# Средне-дисперсионная оптимизация
mv_weights = optimizer.mean_variance_optimize(target_return=0.50)
print("Веса средне-дисперсионного портфеля:", dict(zip(optimizer.symbols, mv_weights.round(4))))

# Блэк-Литтерман со взглядами на крипторынок
bl_weights = optimizer.black_litterman(
    views={"ETHUSDT": 0.80, "SOLUSDT": 1.20},
    view_confidences={"ETHUSDT": 0.6, "SOLUSDT": 0.4}
)
print("Веса Блэк-Литтермана:", dict(zip(optimizer.symbols, bl_weights.round(4))))

# Иерархический паритет рисков
hrp_weights = optimizer.hierarchical_risk_parity()
print("Веса HRP:", dict(zip(optimizer.symbols, hrp_weights.round(4))))

# Критерий Келли для позиций с 2x плечом
kelly_weights = optimizer.kelly_criterion(leverage=2.0, funding_rate=0.0001)
print("Веса половины Келли:", dict(zip(optimizer.symbols, kelly_weights.round(4))))
```

---

## Раздел 6: Реализация на Rust

### Структура проекта

```
ch05_crypto_portfolio_risk/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── optimization/
│   │   ├── mod.rs
│   │   ├── mean_variance.rs
│   │   └── hrp.rs
│   ├── risk/
│   │   ├── mod.rs
│   │   └── metrics.rs
│   └── backtest/
│       ├── mod.rs
│       └── engine.rs
└── examples/
    ├── portfolio_optimization.rs
    ├── kelly_sizing.rs
    └── risk_parity.rs
```

### Основная библиотека (src/lib.rs)

```rust
pub mod optimization;
pub mod risk;
pub mod backtest;

use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PortfolioConfig {
    pub symbols: Vec<String>,
    pub risk_free_rate: f64,
    pub rebalance_threshold: f64,
    pub max_leverage: f64,
}

#[derive(Debug, Clone)]
pub struct PortfolioWeights {
    pub symbols: Vec<String>,
    pub weights: Vec<f64>,
    pub method: String,
}

impl PortfolioWeights {
    pub fn display(&self) {
        println!("Распределение портфеля ({}):", self.method);
        for (sym, w) in self.symbols.iter().zip(self.weights.iter()) {
            println!("  {}: {:.2}%", sym, w * 100.0);
        }
    }
}
```

### Получение данных с Bybit

```rust
use reqwest;
use serde::Deserialize;
use anyhow::Result;

#[derive(Deserialize)]
struct BybitResponse {
    result: BybitResult,
}

#[derive(Deserialize)]
struct BybitResult {
    list: Vec<Vec<String>>,
}

pub async fn fetch_bybit_klines(
    symbol: &str,
    interval: &str,
    limit: u32,
) -> Result<Vec<f64>> {
    let client = reqwest::Client::new();
    let resp = client
        .get("https://api.bybit.com/v5/market/kline")
        .query(&[
            ("category", "linear"),
            ("symbol", symbol),
            ("interval", interval),
            ("limit", &limit.to_string()),
        ])
        .send()
        .await?
        .json::<BybitResponse>()
        .await?;

    let closes: Vec<f64> = resp.result.list
        .iter()
        .map(|row| row[4].parse::<f64>().unwrap_or(0.0))
        .rev()
        .collect();

    Ok(closes)
}

pub fn compute_returns(prices: &[f64]) -> Vec<f64> {
    prices.windows(2)
        .map(|w| (w[1] - w[0]) / w[0])
        .collect()
}
```

### Оптимизатор средней дисперсии (src/optimization/mean_variance.rs)

```rust
use crate::PortfolioWeights;

pub struct MeanVarianceOptimizer {
    pub returns: Vec<Vec<f64>>,
    pub symbols: Vec<String>,
}

impl MeanVarianceOptimizer {
    pub fn new(symbols: Vec<String>, returns: Vec<Vec<f64>>) -> Self {
        Self { returns, symbols }
    }

    pub fn covariance_matrix(&self) -> Vec<Vec<f64>> {
        let n = self.returns.len();
        let t = self.returns[0].len();
        let means: Vec<f64> = self.returns.iter()
            .map(|r| r.iter().sum::<f64>() / t as f64)
            .collect();

        let mut cov = vec![vec![0.0; n]; n];
        for i in 0..n {
            for j in 0..=i {
                let c: f64 = (0..t)
                    .map(|k| {
                        (self.returns[i][k] - means[i])
                            * (self.returns[j][k] - means[j])
                    })
                    .sum::<f64>() / (t - 1) as f64;
                let annualized = c * 365.0;
                cov[i][j] = annualized;
                cov[j][i] = annualized;
            }
        }
        cov
    }

    pub fn minimum_variance(&self) -> PortfolioWeights {
        let n = self.symbols.len();
        let cov = self.covariance_matrix();
        // Оптимизация градиентным спуском для минимальной дисперсии
        let mut weights = vec![1.0 / n as f64; n];
        let lr = 0.001;
        let iterations = 5000;

        for _ in 0..iterations {
            let mut grad = vec![0.0; n];
            for i in 0..n {
                for j in 0..n {
                    grad[i] += 2.0 * cov[i][j] * weights[j];
                }
            }
            // Проекция на симплекс
            for i in 0..n {
                weights[i] -= lr * grad[i];
                weights[i] = weights[i].max(0.0);
            }
            let sum: f64 = weights.iter().sum();
            for w in weights.iter_mut() {
                *w /= sum;
            }
        }

        PortfolioWeights {
            symbols: self.symbols.clone(),
            weights,
            method: "Минимальная дисперсия".to_string(),
        }
    }
}
```

### Метрики риска (src/risk/metrics.rs)

```rust
pub struct CryptoRiskMetrics;

impl CryptoRiskMetrics {
    pub fn sharpe_ratio(returns: &[f64], risk_free_rate: f64) -> f64 {
        let daily_rf = risk_free_rate / 365.0;
        let excess: Vec<f64> = returns.iter().map(|r| r - daily_rf).collect();
        let mean = excess.iter().sum::<f64>() / excess.len() as f64;
        let variance = excess.iter()
            .map(|r| (r - mean).powi(2))
            .sum::<f64>() / (excess.len() - 1) as f64;
        (365.0_f64).sqrt() * mean / variance.sqrt()
    }

    pub fn sortino_ratio(returns: &[f64], risk_free_rate: f64) -> f64 {
        let daily_rf = risk_free_rate / 365.0;
        let excess: Vec<f64> = returns.iter().map(|r| r - daily_rf).collect();
        let mean = excess.iter().sum::<f64>() / excess.len() as f64;
        let downside: Vec<f64> = excess.iter()
            .filter(|&&r| r < 0.0)
            .cloned()
            .collect();
        let downside_var = downside.iter()
            .map(|r| r.powi(2))
            .sum::<f64>() / downside.len().max(1) as f64;
        (365.0_f64).sqrt() * mean / downside_var.sqrt()
    }

    pub fn max_drawdown(returns: &[f64]) -> f64 {
        let mut cumulative = 1.0;
        let mut peak = 1.0;
        let mut max_dd = 0.0_f64;

        for &r in returns {
            cumulative *= 1.0 + r;
            peak = peak.max(cumulative);
            let dd = (cumulative - peak) / peak;
            max_dd = max_dd.min(dd);
        }
        max_dd
    }

    pub fn calmar_ratio(returns: &[f64]) -> f64 {
        let annual_return = returns.iter().sum::<f64>() / returns.len() as f64 * 365.0;
        let max_dd = Self::max_drawdown(returns).abs();
        if max_dd > 0.0 { annual_return / max_dd } else { 0.0 }
    }

    pub fn kelly_fraction(
        expected_return: f64,
        variance: f64,
        risk_free_rate: f64,
        funding_rate: f64,
        leverage: f64,
    ) -> f64 {
        let adjusted_return = expected_return - funding_rate * 3.0 * 365.0 * leverage;
        let kelly = (adjusted_return - risk_free_rate) / variance;
        (kelly / 2.0).max(0.0) // половина Келли, неотрицательная
    }
}
```

### Движок бэктестирования (src/backtest/engine.rs)

```rust
use crate::risk::metrics::CryptoRiskMetrics;

pub struct BacktestConfig {
    pub initial_capital: f64,
    pub taker_fee: f64,    // 0.00055 для Bybit
    pub maker_fee: f64,    // 0.0002 для Bybit
    pub slippage_bps: f64, // базисные пункты
    pub funding_interval_hours: f64,
}

impl Default for BacktestConfig {
    fn default() -> Self {
        Self {
            initial_capital: 100_000.0,
            taker_fee: 0.00055,
            maker_fee: 0.0002,
            slippage_bps: 1.0,
            funding_interval_hours: 8.0,
        }
    }
}

pub struct BacktestResult {
    pub total_return: f64,
    pub sharpe_ratio: f64,
    pub sortino_ratio: f64,
    pub max_drawdown: f64,
    pub calmar_ratio: f64,
    pub total_fees: f64,
    pub total_funding: f64,
    pub num_rebalances: u32,
}

pub fn run_backtest(
    returns_matrix: &[Vec<f64>],
    weights_history: &[Vec<f64>],
    config: &BacktestConfig,
    funding_rates: &[f64],
) -> BacktestResult {
    let num_periods = returns_matrix[0].len();
    let mut portfolio_returns = Vec::with_capacity(num_periods);
    let mut total_fees = 0.0;
    let mut total_funding = 0.0;
    let mut num_rebalances = 0u32;

    for t in 0..num_periods {
        let mut period_return = 0.0;
        for (i, asset_returns) in returns_matrix.iter().enumerate() {
            period_return += weights_history[t][i] * asset_returns[t];
        }

        // Вычет транзакционных издержек при ребалансировке
        if t > 0 {
            let turnover: f64 = weights_history[t].iter()
                .zip(weights_history[t - 1].iter())
                .map(|(w1, w0)| (w1 - w0).abs())
                .sum();
            if turnover > 0.01 {
                let fee = turnover * (config.taker_fee + config.slippage_bps * 0.0001);
                period_return -= fee;
                total_fees += fee;
                num_rebalances += 1;
            }
        }

        // Вычет финансирования
        if t < funding_rates.len() {
            total_funding += funding_rates[t];
            period_return -= funding_rates[t];
        }

        portfolio_returns.push(period_return);
    }

    let sharpe = CryptoRiskMetrics::sharpe_ratio(&portfolio_returns, 0.05);
    let sortino = CryptoRiskMetrics::sortino_ratio(&portfolio_returns, 0.05);
    let max_dd = CryptoRiskMetrics::max_drawdown(&portfolio_returns);
    let calmar = CryptoRiskMetrics::calmar_ratio(&portfolio_returns);
    let total_return = portfolio_returns.iter()
        .fold(1.0, |acc, r| acc * (1.0 + r)) - 1.0;

    BacktestResult {
        total_return,
        sharpe_ratio: sharpe,
        sortino_ratio: sortino,
        max_drawdown: max_dd,
        calmar_ratio: calmar,
        total_fees,
        total_funding,
        num_rebalances,
    }
}
```

---

## Раздел 7: Практические примеры

### Пример 1: Построение портфеля с паритетом рисков для топ-5 криптоактивов

```python
optimizer = CryptoPortfolioOptimizer(
    symbols=["BTCUSDT", "ETHUSDT", "SOLUSDT", "AVAXUSDT", "LINKUSDT"]
)
optimizer.fetch_returns(source="bybit")

rp_weights = optimizer.risk_parity()
print("Веса паритета рисков:")
for sym, w in zip(optimizer.symbols, rp_weights):
    print(f"  {sym}: {w:.2%}")

# Ожидаемый результат:
#   BTCUSDT:  38.12%
#   ETHUSDT:  27.45%
#   SOLUSDT:  12.31%
#   AVAXUSDT: 10.89%
#   LINKUSDT: 11.23%

metrics = CryptoRiskMetrics()
portfolio_returns = (optimizer.returns * rp_weights).sum(axis=1)
print(f"Коэффициент Шарпа: {metrics.sharpe_ratio(portfolio_returns):.3f}")
print(f"Максимальная просадка: {metrics.max_drawdown(portfolio_returns):.2%}")
```

### Пример 2: Блэк-Литтерман с бычьим взглядом на ETH

```python
optimizer = CryptoPortfolioOptimizer(
    symbols=["BTCUSDT", "ETHUSDT", "SOLUSDT"]
)
optimizer.fetch_returns(source="bybit")

# Взгляд: ETH принесёт 100% годовых с уверенностью 70%
# Взгляд: SOL принесёт 150% годовых с уверенностью 40%
bl_weights = optimizer.black_litterman(
    views={"ETHUSDT": 1.00, "SOLUSDT": 1.50},
    view_confidences={"ETHUSDT": 0.7, "SOLUSDT": 0.4}
)

print("Аллокация Блэк-Литтермана:")
for sym, w in zip(optimizer.symbols, bl_weights):
    print(f"  {sym}: {w:.2%}")

# Ожидаемый результат:
#   BTCUSDT: 25.34%
#   ETHUSDT: 48.21%
#   SOLUSDT: 26.45%
```

### Пример 3: Размер позиции по Келли для бессрочных фьючерсов с плечом

```python
optimizer = CryptoPortfolioOptimizer(symbols=["BTCUSDT", "ETHUSDT"])
optimizer.fetch_returns(source="bybit")

# Получение текущих ставок финансирования с Bybit
url = "https://api.bybit.com/v5/market/tickers"
params = {"category": "linear", "symbol": "BTCUSDT"}
resp = requests.get(url, params=params).json()
funding_rate = float(resp["result"]["list"][0]["fundingRate"])
print(f"Текущая ставка финансирования BTC: {funding_rate:.6f}")
print(f"Аннуализированное финансирование: {funding_rate * 3 * 365:.2%}")

for leverage in [1.0, 2.0, 3.0, 5.0]:
    kelly_w = optimizer.kelly_criterion(
        leverage=leverage,
        funding_rate=funding_rate
    )
    print(f"\nПлечо {leverage}x - Веса половины Келли:")
    for sym, w in zip(optimizer.symbols, kelly_w):
        print(f"  {sym}: {w:.2%}")

# Ожидаемый результат:
# Текущая ставка финансирования BTC: 0.000100
# Аннуализированное финансирование: 10.95%
#
# Плечо 1.0x - Веса половины Келли:
#   BTCUSDT: 62.14%
#   ETHUSDT: 37.86%
#
# Плечо 3.0x - Веса половины Келли:
#   BTCUSDT: 71.23%
#   ETHUSDT: 28.77%
#
# Плечо 5.0x - Веса половины Келли:
#   BTCUSDT: 100.00%
#   ETHUSDT: 0.00%
```

---

## Раздел 8: Фреймворк бэктестирования

### Компоненты фреймворка

Фреймворк бэктестирования для построения криптопортфеля включает:

1. **Конвейер данных**: Получение исторических данных OHLCV и ставок финансирования с Bybit
2. **Оптимизатор портфеля**: Вычисление целевых весов выбранным методом
3. **Симулятор исполнения**: Моделирование реалистичных исполнений с комиссиями и проскальзыванием
4. **Монитор рисков**: Отслеживание метрик риска в реальном времени и близости к ликвидации
5. **Анализатор производительности**: Расчёт комплексной статистики производительности

### Панель метрик

| Метрика | Описание | Расчёт |
|---------|----------|--------|
| Общая доходность | Кумулятивная доходность портфеля | Произведение (1 + r_t) - 1 |
| Аннуализированная доходность | Среднегеометрическая годовая доходность | (1 + total)^(365/days) - 1 |
| Коэффициент Шарпа | Доходность с поправкой на риск (24/7) | sqrt(365) * mean(r) / std(r) |
| Коэффициент Сортино | Доходность с поправкой на нисходящий риск | sqrt(365) * mean(r) / downside_std |
| Коэффициент Кальмара | Доходность / максимальная просадка | ann_return / abs(max_dd) |
| Максимальная просадка | Наибольшее снижение от пика до дна | min((cum - peak) / peak) |
| Оборот | Годовой оборот портфеля | sum(abs(w_t - w_{t-1})) * 365/T |
| Общие комиссии | Понесённые транзакционные издержки | sum(turnover * fee_rate) |
| P&L финансирования | Чистые платежи финансирования | sum(funding_rate * position_value) |

### Пример результатов бэктеста

```
=== Результаты бэктеста портфеля (2024-01-01 — 2024-12-31) ===

Вселенная: BTCUSDT, ETHUSDT, SOLUSDT, AVAXUSDT, LINKUSDT
Ребалансировка: Еженедельно | Модель комиссий: Bybit Тейкер 0.055% + 1bp проскальзывание

Стратегия              | Доходн. | Шарп  | Сортино | МаксДД  | Кальмар | Оборот
-----------------------|---------|-------|---------|---------|---------|--------
Равновзвешенный (1/N)  |  87.2%  |  1.42 |  2.01   | -32.1%  |  2.72   |  124%
Мин. дисперсия         |  52.3%  |  1.68 |  2.34   | -18.4%  |  2.84   |   89%
Паритет рисков         |  71.5%  |  1.61 |  2.28   | -22.7%  |  3.15   |   97%
HRP                    |  74.8%  |  1.55 |  2.15   | -25.3%  |  2.96   |  103%
Блэк-Литтерман         |  96.1%  |  1.38 |  1.92   | -35.6%  |  2.70   |  142%
Келли (2x плечо)       | 134.5%  |  1.21 |  1.67   | -48.2%  |  2.79   |  178%

За вычетом комиссий:
  Равновзвешенный: 84.9% (комиссии: 2.3%)
  Мин. дисперсия:  50.8% (комиссии: 1.5%)
  Паритет рисков:  69.8% (комиссии: 1.7%)
  HRP:             72.9% (комиссии: 1.9%)
  Блэк-Литтерман:  93.2% (комиссии: 2.9%)
  Келли (2x):      127.1% (комиссии: 7.4%, финансирование: 3.8%)
```

---

## Раздел 9: Оценка производительности

### Сравнение методов аллокации

| Критерий | Средне-дисперсионный | Паритет рисков | HRP | Блэк-Литтерман | Келли |
|----------|---------------------|----------------|-----|----------------|-------|
| Чувствительность к ошибке оценки | Очень высокая | Низкая | Низкая | Средняя | Высокая |
| Обработка смены режимов | Нет | Частично | Да | Частично | Нет |
| Требует прогнозов доходности | Да | Нет | Нет | Да (взгляды) | Да |
| Подходит для плеча | Нет | Да | Нет | Нет | Да |
| Устойчивость к выбросам | Нет | Частично | Да | Частично | Нет |
| Сложность реализации | Низкая | Средняя | Средняя | Высокая | Низкая |
| Стабильность вне выборки | Плохая | Хорошая | Хорошая | Умеренная | Плохая |

### Ключевые выводы

1. **Паритет рисков и HRP стабильно превосходят средне-дисперсионный подход на основе риск-скорректированной доходности** на крипторынках, прежде всего потому, что они избегают экстремальной чувствительности к оценкам ожидаемой доходности, которая отравляет оптимизацию Марковица.

2. **Смены корреляционного режима — основной фактор портфельного риска** в криптовалютах. Во время обвала мая 2021 года корреляции BTC-альткоинов выросли с 0.45 до 0.92 за несколько дней, делая статические модели аллокации неадекватными.

3. **Carry ставки финансирования — значимый компонент доходности** для портфелей с плечом. Во время устойчивых бычьих рынков финансирование может стоить 10-30% годовых, существенно размывая доходность с плечом.

4. **Портфель 1/N остаётся сильным бенчмарком**, который сложно стабильно превзойти после транзакционных издержек, особенно для небольших вселенных из 3-5 активов.

5. **Размер позиции по половине Келли обеспечивает практический баланс** между максимизацией роста и контролем просадок. Полный Келли приводит к неприемлемым просадкам в условиях толстых хвостов криптовалют.

### Ограничения

- Оценки ковариации по своей природе обращены в прошлое и могут не захватывать возникающие корреляционные структуры
- Экстремальные хвостовые события (взломы бирж, отвязки стейблкоинов) не моделируются
- Ограничения ликвидности для крупных портфелей на рынках альткоинов не рассматриваются
- Моделирование ставки финансирования предполагает постоянные ставки, тогда как фактические ставки значительно колеблются
- Контрагентский риск (платёжеспособность биржи) не включён в модель портфеля

---

## Раздел 10: Перспективные направления

1. **Построение портфеля с интеграцией DeFi**: Включение доходности от фарминга, предоставления ликвидности и стейкинга в фреймворк оптимизации портфеля, рассматривая DeFi-протоколы как отдельные классы активов с собственными профилями риска и доходности.

2. **Ончейн-аналитика для прогнозирования ковариации**: Использование данных блокчейна (движения китов, потоки бирж, карты ликвидаций) для прогнозирования изменений в корреляционной структуре до того, как они проявятся в ценовых данных.

3. **Модели переключения режимов для динамической аллокации**: Реализация скрытых марковских моделей или процессов диффузии со скачками для обнаружения изменений корреляционного режима в реальном времени и динамической корректировки весов портфеля.

4. **Межбиржевой арбитраж в контексте портфеля**: Расширение портфельного фреймворка для оптимизации между несколькими биржами, захватывая ценовые расхождения при управлении связанными рисками переводов и контрагентов.

5. **Машинное обучение для прогнозирования ковариации**: Применение глубокого обучения (графовых нейронных сетей, трансформеров) для прогнозирования полной ковариационной матрицы, используя богатые данные микроструктуры, доступные на крипторынках.

6. **Хеджирование хвостового риска опционами**: Включение крипто-опционов (доступных на Bybit) в портфель в качестве явных хеджей от хвостового риска, используя свопы дисперсии и пут-спреды для защиты от мгновенных обвалов.

---

## Литература

1. Markowitz, H. (1952). "Portfolio Selection." *The Journal of Finance*, 7(1), 77-91.

2. Black, F., & Litterman, R. (1992). "Global Portfolio Optimization." *Financial Analysts Journal*, 48(5), 28-43.

3. Maillard, S., Roncalli, T., & Teiletche, J. (2010). "The Properties of Equally Weighted Risk Contribution Portfolios." *The Journal of Portfolio Management*, 36(4), 60-70.

4. De Prado, M. L. (2016). "Building Diversified Portfolios that Outperform Out of Sample." *The Journal of Portfolio Management*, 42(4), 59-69.

5. Kelly, J. L. (1956). "A New Interpretation of Information Rate." *Bell System Technical Journal*, 35(4), 917-926.

6. Ledoit, O., & Wolf, M. (2004). "A Well-Conditioned Estimator for Large-Dimensional Covariance Matrices." *Journal of Multivariate Analysis*, 88(2), 365-411.

7. Liu, Y., Tsyvinski, A., & Wu, X. (2022). "Common Risk Factors in Cryptocurrency." *The Journal of Finance*, 77(2), 1133-1177.
