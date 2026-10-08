# Descriptive Statistics

## Математическое ожидание

$$
E[X] = \sum_{i=1}^{n} x_i P(x_i)
$$

## Дисперсия

$$
Var(X) = E[(X - E[X])^2]
$$

## Стандартное отклонение

$$
Std(X) = \sqrt{Var(X)}
$$

## Ковариация

$$
Cov(X, Y) = E[(X - E[X])(Y - E[Y])]
$$

## Корреляция

$$
Cor(X, Y) = \frac{Cov(X, Y)}{\sqrt{Var(X) Var(Y)}}
$$

## Если спросят

- **Зачем:** определения $E$, $Var$, $Cov$, $Cor$ — из них собираются шум и Gauss-Markov.
- **Когда ломается:** корреляцию считают как ковариацию, забыв нормировку на $\mathrm{std}$.
- **Путают с:** выборочными моментами vs теоретическими.
- **Фраза:** корреляция — нормированная ковариация.

## Связанное

- [[Error types]]
- [[Gauss Markov Theorem]]
- [[Bias Variance Tradeoff]]
- [[Linear regression model]]
- [[Bias Variance Tradeoff]]

