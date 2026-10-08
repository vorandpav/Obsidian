# Prior And Posterior Distribution

### Prior and posterior distributions

#### Идея
В Байесовской статистике параметры модели $\theta$ рассматриваются как **случайные величины**. До проведения эксперимента мы задаем **априорное распределение (Prior)**, отражающее наши начальные знания. После наблюдения данных $D$ мы обновляем представление о параметрах и получаем **апостериорное распределение (Posterior)**.

#### Формулы
Основное соотношение теории Байеса:
$$P(\theta \mid D) = \frac{P(D \mid \theta) P(\theta)}{P(D)}$$

Или в виде пропорциональности (без нормировочной константы $P(D)$):
$$P(\theta \mid D) \propto P(D \mid \theta) \cdot P(\theta)$$
$$\text{Posterior} \propto \text{Likelihood} \times \text{Prior}$$

##### Сопряженные распределения (Conjugate Priors)
Если Prior и Posterior принадлежат к одному семейству распределений, то Prior называется *сопряженным* к Likelihood.
* **Beta-Binomial:** Prior $\text{Beta}(\alpha, \beta)$ + Likelihood $\text{Binomial}(n, k) \rightarrow$ Posterior $\text{Beta}(\alpha + k, \beta + n - k)$.
* **Normal-Normal:** Prior $\mathcal{N}(\mu_0, \sigma_0^2)$ + Likelihood $\mathcal{N}(\mu, \sigma^2) \rightarrow$ Posterior $\mathcal{N}(\mu_{post}, \sigma_{post}^2)$.
* **Dirichlet-Multinomial:** Обобщение Beta-Binomial на многомерный случай.

---

#### Примеры

##### Пример 1: Оценка честности монеты (Beta-Binomial)
Пусть $\theta$ — вероятность выпадения орла.

1. **Prior:** Мы полагаем, что монета скорее всего честная, но сомневаемся. Задаем Prior в виде $\text{Beta}(\alpha=2, \beta=2)$.
   $$\mathbb{E}[\theta] = \frac{2}{2+2} = 0.5$$

2. **Данные ($D$):** Сделали $10$ бросков: получили $k = 8$ орлов и $2$ решки.

3. **Likelihood:** $P(D \mid \theta) \propto \theta^8 (1-\theta)^2$.

4. **Posterior:**
   $$P(\theta \mid D) \propto \theta^{2-1}(1-\theta)^{2-1} \cdot \theta^8 (1-\theta)^2 = \theta^9 (1-\theta)^3$$
   Получили $\text{Beta}(\alpha_{post}=10, \beta_{post}=4)$.
   $$\mathbb{E}[\theta \mid D] = \frac{10}{10+4} \approx 0.714$$

*Сравнение:* Частотная оценка (MLE) дала бы $\frac{8}{10} = 0.8$. Апостериорное математическое ожидание ($0.714$) оказалось ближе к $0.5$, так как Prior «сгладил» аномально высокую частоту орлов на малой выборке.

##### Пример 2: Оценка CTR нового баннера при малом числе кликов
* **Prior:** Исторический CTR всех баннеров в системе распределен как $\text{Beta}(5, 95)$ (средний CTR = $5\%$).
* **Данные ($D$):** Новый баннер показали $10$ раз, кликнули $3$ раза.
* **Без Байеса (MLE):** $\text{CTR} = \frac{3}{10} = 30\%$ (оценка завышена из-за малой выборки).
* **С Байесом (Posterior):** $\text{Beta}(5 + 3, 95 + 7) = \text{Beta}(8, 102)$.
  $$\mathbb{E}[\text{CTR} \mid D] = \frac{8}{8 + 102} = \frac{8}{110} \approx 7.27\%$$
  Модель не делает поспешных выводов о $30\%$ CTR на основе всего 10 показов.

---

#### Если спросят
* **Зачем:** Позволяет корректно учитывать априорные знания, сглаживать оценки на малых выборках и оценивать неопределенность (ширину апостериорного распределения).
* **Когда ломается:** При неудачно выбранном слишком «строгом» Prior (вызывает сильное смещение); если $P(D)$ не вычисляется аналитически и данные многомерны (требуются методы MCMC / Variational Inference).
* **Путают с:** MLE (MLE находит только точечный максимум Likelihood и не использует Prior).
* **Фраза:** Posterior — это компромисс между априорным знанием (Prior) и данными (Likelihood); при $N \to \infty$ влияние Prior исчезает.

#### Связанное
* [[Likelihood]]
* [[Maximum Likelihood Estimation]]
* [[Naive Bayes classifier]]
* [[Statistical tests]]
* [[Descriptive statistics]]

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-

