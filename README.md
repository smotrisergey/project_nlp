
# Исследование контроля количества слов языковых моделией

Авторы [статьи](https://web.stanford.edu/class/archive/cs/cs224n/cs224n.1244/final-projects/KatherineLi.pdf) утверждают, что обученная с помощью добавления специальных токенов длины предложения  модель (125 М параметров) значительно превосходит такие модели, как ChatGPT и GPT-2 Large в генерации предложений заданной длины, несмотря на более малый размер. Добавление WCC токенов не испортило качество генерируемого текста. В нашей работы мы попытались воспроизвести результаты, описанные в статье, и применить данную методику к генерации абзацев заданной длины.

# Датасет

[Датасет Openwebtext](https://huggingface.co/datasets/Skylion007/openwebtext) (350k - from scratch, 10k - fine-tuning)
Разбиение на предложения и подсчет слов с помощью stanza
Разбиваем на предложения и в начале каждого предложения вставляем “токен длины” - <N+1>. Где N – длина предложения
Далее рассматриваем 3 подхода:
1. Добавление WCC токенов к предложениям без дальнейшей конкатенации
2. Добавление WCC токенов к предложением с конкатенацией
3. Добавление одного WCC токена к абзацу
Полное создание датасета заняло около 3 часов на 2-х GPU A6000

# Архитектуры для fine-tuning

1. GPT-2 Large - (context = 1024), была использована в статье
2. llama-2-7b-hf-small - (context = 4096), open-source модель с большим контекстным окном, самая большая модель, на которой проводились эксперименты 
3. LaMini-Neo-125M - (context = 4096), небольшая open-source модель, использовалась для быстрого обучения в условиях ограниченных вычислительных ресурсах
4. mamba-130m-hf - модель, основанная не на трансформерной архитектуре, использовалась для расширения интерпретации выводов, полученных в исходной статье, о свойствах трансформерных моделей планировать количество слов. 
5. TinyLlama-1.1B-Chat-v1.0 - средняя open-source модель, использовалась для долгого обучения на арендованной видеокарте

# Результаты

Нам удалось воспроизвести результаты в статьи только в одном эксперименте:
* GPT Neo, 125M параметров, контекстное окно – 2048
* Датасет с конкатенированными предложениями с токенами длин в начале каждого
* Обучение с нуля
* 1*A100 GPU, 128Gb RAM
* Обучение 2 часа
* Flash Attention 2, Deep Speed (для оптимизации под А100)
* Float16 dtype

![text](https://post-images.org/photo-page.php?photo=uYJhskbp) 
