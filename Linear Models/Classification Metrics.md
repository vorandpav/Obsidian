# Classification Metrics

* TP - True Positive
* TN - True Negative
* FP - False Positive - ошибка 1 рода
* FN - False Negative - ошибка 2 рода
[[Error Types]]


## Accuracy

$$
Accuracy = \frac{TP + TN}{TP + TN + FP + FN}
$$

Возникает проблема с несбалансированными классами.

## Balanced Accuracy

$$
Balanced Accuracy = \frac{1}{C} \sum_{c=1}^{C} \frac{\sum_{i=1}^{n} \mathbb{1}(y^{(i)} = c \text{ and } \hat{y}^{(i)} = c)}{\sum_{i=1}^{n} \mathbb{1}(y^{(i)} = c)}
$$


## Precision

$$
Precision = \frac{TP}{TP + FP}
$$

Насколько мы точны в классификации, когда мы классифицируем как 1. Пример: пропуск людей на режимный объект (лучше не пустить 100 своих, чем пропустить 1 чужого)


## Recall

$$
Recall = \frac{TP}{TP + FN}
$$

Какую долю объектов класса 1 мы классифицируем как 1. Пример: поиск вирусов в системе (лучше проверить 100 не зараженных, чем не проверить 1 зараженного)

## F1 Score

$$
F1_{Score} = 2 \cdot \frac{Precision \cdot Recall}{Precision + Recall}
$$

Среднее гармоническое между precision и recall. Позволяет сбалансировать precision и recall.

![[Pasted image 20260809181612.png]]

$$
F1_{\beta} = (1 + \beta^2) \cdot \frac{Precision \cdot Recall}{(\beta^2 \cdot Precision) + Recall}
$$

Позволяет сбалансировать precision и recall. $\beta$ - параметр, который позволяет большее внимание уделять precision или recall. Если $\beta > 1$, то большее внимание уделяется precision, если $\beta < 1$, то большее внимание уделяется recall.

## Receiver Operating Characteristic (ROC) Curve
True Positive Rate (TPR) - это доля объектов класса 1, которые мы классифицируем как 1.

$$
TPR = \frac{TP}{TP + FN}
$$

False Positive Rate (FPR) - это доля объектов класса 0, которые мы классифицируем как 1.

$$
FPR = \frac{FP}{FP + TN}
$$

Смысл кривой в том, что мы перебираем все возможные пороги и строим график зависимости TPR от FPR. В точке (0, 0) мы классифицируем все объекты как 0, а значит TPR = 0 и FPR = 0. В точке (1, 1) мы классифицируем все объекты как 1, а значит TPR = 1 и FPR = 1. Каждый раз, когда мы увеличиваем порог, либо TPR увеличивается, либо FPR увеличивается.

## Area Under the Curve (AUC)
AUC - это площадь под ROC кривой. Позволяет оценить качество модели.

$$
AUC = \int_{0}^{1} TPR(t) \cdot FPR(t) \cdot dt
$$

![[Pasted image 20260809205438.png]]


## Когда что важнее

| Сценарий | Важнее | Почему |
|----------|--------|--------|
| Спам-фильтр | Precision | FP = важное письмо в спам |
| Диагностика рака | Recall | FN = пропустили болезнь |
| Сбалансированные классы | Accuracy / F1 | Ок |
| Несбалансированные (1% fraud) | F1, PR-AUC | Accuracy обманчива |

## PR-AUC

При сильном дисбалансе смотри **PR-AUC**, не только ROC-AUC (Pos < ~5%).

## Порог классификации

По умолчанию $\hat{y} \geq 0.5$. Порог подбирают по F1 / бизнес-метрике на validation.

## Если спросят

- **Зачем:** accuracy врёт на дисбалансе; precision — точность плюсов, recall — полнота.
- **Когда ломается:** редкий класс + accuracy; ROC при сильном дисбалансе плохо отражает редкий плюс.
- **Путают с:** TPR и precision: в знаменателе разные множества.
- **Фраза:** precision — из названных единицами; recall — какую долю единиц нашли.

## Связанное

- [[Error Types]]
- [[Multiclass Metrics]]
- [[Logistic Regression]]
- [[Multiclass Classification]]
