# Исследование контроля количества слов языковых моделией

## Введение

Авторы [статьи](https://web.stanford.edu/class/archive/cs/cs224n/cs224n.1244/final-projects/KatherineLi.pdf) утверждают, что обученная с помощью добавления специальных токенов длины предложения  модель (125 М параметров) значительно превосходит такие модели, как ChatGPT и GPT-2 Large в генерации предложений заданной длины, несмотря на более малый размер. Добавление WCC токенов не испортило качество генерируемого текста. В нашей работы мы попытались воспроизвести результаты, описанные в статье, и применить данную методику к генерации абзацев заданной длины.

## Датасет

[Датасет Openwebtext](https://huggingface.co/datasets/Skylion007/openwebtext) (350k - from scratch, 10k - fine-tuning)
Разбиение на предложения и подсчет слов с помощью stanza
Разбиваем на предложения и в начале каждого предложения вставляем “токен длины” - <N+1>. Где N – длина предложения
Далее рассматриваем 3 подхода:
1. Добавление WCC токенов к предложениям без дальнейшей конкатенации
2. Добавление WCC токенов к предложением с конкатенацией
3. Добавление одного WCC токена к абзацу
Полное создание датасета заняло около 3 часов на 2-х GPU A6000

## Архитектуры для fine-tuning

1. GPT-2 Large - (context = 1024), была использована в статье
2. llama-2-7b-hf-small - (context = 4096), open-source модель с большим контекстным окном, самая большая модель, на которой проводились эксперименты 
3. LaMini-Neo-125M - (context = 4096), небольшая open-source модель, использовалась для быстрого обучения в условиях ограниченных вычислительных ресурсах
4. mamba-130m-hf - модель, основанная не на трансформерной архитектуре, использовалась для расширения интерпретации выводов, полученных в исходной статье, о свойствах трансформерных моделей планировать количество слов. 
5. TinyLlama-1.1B-Chat-v1.0 - средняя open-source модель, использовалась для долгого обучения на арендованной видеокарте

## Результаты

Нам удалось воспроизвести результаты в статьи только в одном эксперименте:
* GPT Neo, 125M параметров, контекстное окно – 2048
* Датасет с конкатенированными предложениями с токенами длин в начале каждого
* Обучение с нуля (2 часа)
* 1*A100 GPU, 128Gb RAM
* Flash Attention 2, Deep Speed (для оптимизации под А100)
* Float16 dtype

![Результат](https://github.com/smotrisergey/project_nlp/blob/main/result.png)

средний mse  - **6.74**  
средний accuracy - **0.75**

[Веса модели, обученной с нуля](https://drive.google.com/file/d/15ojn7E0W61Uo6vyAfoQkvPROwTaqlb_P/view?usp=drive_link) (лучший результат)  
[Веса модели, дообученной](https://drive.google.com/file/d/1dODccR9GQn_g-mf4kUVHtr_-cTqAlZ3m/view?usp=drive_link) (плохой результат)

## Видеоотчет

[Видеоотчет](https://drive.google.com/file/d/1rdQ9xwqC-lZFveI3fwkGe0hCnv5XUZuK/view) по данному проекту

## Выводы

* Fine-Tuning не показал удовлетворительных результатов, модель не учитывала токен длины предложения/абзаца при генерации текста. Такой результат может быть связан с небольшим количеством данных (10k абзацев, 250k предложений). Также трюки, применяемые для дообучения моделей внесли отрицательный вклад в качество итоговой модели. Данный подход не применим для дообучения моделей (ни предложения, ни абзацы)

* Дообучение моделей не работает, приемлемый результат получился только при обучении с нуля
* Мы не смогли добиться хороших результатов в обучении моделей для генерации абзацев заданных длин (авторы статьи говорят, что модель плохо справляется с генераций длинных предложений. Видимо, по этой причине модель плохо генерирует абзацы нужной длины)
* Из всех датасетов себя хорошо показал вариант с конкатенацией предложений с токенами длины в абзацы

## Участники и роли

**Романенко Ярослав, БИВТ-21-16** - обучение моделей на абзацах (эксперименты с длительным обучением), фильтрация абзацев, тестирование оптимизаторов  
**Шинкаренко Роман, БИВТ-21-17** - сбор датасетов, оптимизация обучения по памяти и скорости, обучение моделей на абзацах, обучение моделей с нуля  
**Мартынов Сергей, БИВТ-21-17** - обучение моделей на абзацах  
**Александров Николай, БИВТ-21-17** - сбор и подготовка датасетов, обучение моделей на предложениях  
**Плешаков Иван, БИВТ-21-16** - обучение моделей на предложениях, подготовка отчета и презентации
