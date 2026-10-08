# Sigmoid

Сигмоида - это функция, которая принимает любое значение и возвращает значение между 0 и 1.

$$
\sigma(x) = \frac{1}{1 + e^{-x}} = \frac{e^x}{1 + e^x}
$$

![[Pasted image 20260809163326.png]]

Свойства сигмоиды:
* $1 - \sigma(x) = \sigma(-x)$
* производная сигмоиды:

$$
\frac{d}{dx} \sigma(x) = \sigma(x) (1 - \sigma(x))
$$

## Если спросят

- **Зачем:** $\mathbb{R}\to(0,1)$, $1-\sigma(x)=\sigma(-x)$, $\sigma'=\sigma(1-\sigma)$.
- **Когда ломается:** насыщение на больших $|x|$ — градиент почти 0.
- **Путают с:** softmax.
- **Фраза:** сигмоида даёт вероятность, производная $\sigma(1-\sigma)$.

## Связанное

- [[Logistic regression]]
- [[Cross Entropy]]
- [[Backpropagation]]
- [[LSTM]]
- [[Basic Loss]]
