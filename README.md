# hse23_hw1

#Создаю ссылки в папке
ls /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
