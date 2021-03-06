# Список функций

## Cache

Модуль для сохранения массивов на Google Диск. По умолчанию, без указания расширения файла, подразумевается `json`. При явном указании поддерживается текстовый формат - `file.txt`

?> В случае отсутствия требуемой функциональности, можете реализовать свои функции через [DriveApp](https://developers.google.com/apps-script/reference/drive).

### append

Записывает данные в файл, добавляя новые данные. Если файла не существует, создает его. 

Аргументы
- (строка) `filename` - имя файла
- (массив) `content` - массив данных для добавления
- (строка) `place` - место присоединения. При `begin` в начало файла. При `end` в конец файла. По умолчанию `end`.
- (число) `limit` - ограничить число элементов массива **после** присоединения новых данных. По умолчанию, выбор **первых** 100 тысяч (sliceFirst). Для постоянного обновления потребуется `place` равный 'begin'.

Пример 1 - Добавить новые треки в начало файла. Ограничить массив 5 тысячами треков.
```js
let tracks = Source.getPlaylistTracks('playlist name', 'id');
Cache.append('myfile.json', tracks, 'begin', 5000);
```

Пример 2 - Добавить треки в конец файла. Лимит 100 тысяч. Значения по умолчанию.
```js
let tracks = Source.getPlaylistTracks('playlist name', 'id');
Cache.append('myfile.json', tracks);
```

### clear

Перезаписывает содержимое файла пустым массивом.

Аргументы
- (строка) `filename` - имя файла

Пример 1 - Очистить файл
```js
Cache.clear('filename.json');
```

### compressArtists

Удаляет излишние данные о исполнителе.

Аргументы
- (массив) `artists` - массив исполнителей.

Пример 1 - Сжать данные о исполнителях и сохранить в файл
```js
let artists = Yandex.getArtists('login');
Cache.compressArtists(artists);
Cache.write('yandex-artists.json', artists);
```

### compressTracks

Удаляет излишние данные о треках. Позволяет существенно сократить объем файла.

Аргументы
- (массив) `tracks` - массив треков.

Пример 1 - Сжать треки и сохранить в файл
```js
let tracks = Source.getPlaylistTracks('playlist name', 'id');
Cache.compressTracks(tracks);
Cache.write('myfile.json', tracks);
```
### copy

Создает копию файла. Возвращает имя созданной копии.

Аргументы
- (строка) `filename` - имя файла

Пример 1 - Создать копию файла и получить его данные
```js
let filename = 'myfile.json';
filename = Cache.copy(filename);
let tracks = Cache.read(filename);
```

### read

Возвращает данные из файла. Если файла не существует для `json` вернется пустой массив, для остальных пустая строка.

!> При чтении пустого файла выбрасывается исключение, чтобы предотвратить перезапись файла при баге со стороны Google ([подробнее](https://github.com/Chimildic/goofy/discussions/26)).

Аргументы
- (строка) `filename` - имя файла

Пример 1 - Добавить треки из файла в плейлист
```js
let tracks = Cache.read('file.json');
Playlist.saveAsNew({
    name: 'Треки из файла',
    tracks: tracks,
});
```

### remove

Переносит файл в корзину. По правилам Google Диска, объекты в корзине удаляются через 30 дней.

Аргументы
- (строка) `filename` - имя файла

Пример 1 - Поместить файл в корзину
```js
Cache.remove('filename.json');
```

### rename

Переименовывает файл.

Аргументы
- (строка) `oldFilename` - текущее имя файла
- (строка) `newFilename` - новое имя файла

!> Не используйте имена `SpotifyRecentTracks`, `LastfmRecentTracks`, `BothRecentTracks`. Они используются в механизме накопления [истории прослушиваний](/overview?id=История-прослушиваний).

Пример 1 - Переименовать файл
```js
Cache.rename('filename.json', 'newname.json');
```

### write

Записывает данные в файл. Если файла не существует, создает его. Если файл есть, перезаписывает содержимое.

Аргументы
- (строка) `filename` - имя файла
- (массив) `content` - массив данных для записи

Пример 1 - Записать любимые треки в файл
```js
let tracks = Sourct.getSavedTracks();
Cache.write('liked.json', tracks);
```

## Combiner

Объединение треков разных источников 

### alternate

Возвращает новый массив, в котором чередуются элементы массивов источников.

Аргументы
- (строка) bound - допустимое значение `max` или `min`.
  - Если элементы в одном из массивов закончились, добавляет оставшиеся элементы другого массива в конец (`max`). При нескольких оставшихся массивах, продолжает чередовать их элементы.
  - Если хотя бы в одном массиве закончились элементы, немедленно возвращает результат (`min`). Оставшиеся элементы отбрасываются.
- (перечень массивов) `...arrays` - перечень массивов, чьи элементы необходимо чередовать.

Пример 1 - Чередовать элементы трех массивов.
```js
let firstArray = [1, 3, 5];
let secondeArray = [2, 4, 6, 8, 10];
let thirdArray = [100, 200, 300];
let resultArray = Combiner.alternate('max', firstArray, secondeArray, thirdArray);
// результат 1, 2, 100, 3, 4, 200, 5, 6, 300, 8, 10
```

Пример 2 - Чередовать топ прослушиваний за месяц и любимые треки.
```js
let topTracks = Source.getTopTracks('short'); // допустим, 50 треков
let savedTracks = Source.getSavedTracks(20); //допустим, 20 треков
let resultArray = Combiner.alternate('min', topTracks, savedTracks);
// результат содержит 40 треков
```

### mixin

Возвращает новый массив, в котором чередуются элементы двух массивов в заданной пропорции. Внутри вызывается [mixinMulti](/func?id=mixinmulti) с двумя массивами.

Аргументы
- (массив) `xArray` - первый массив источник.
- (массив) `yArray` - второй массив источник.
- (число) `xRow` - количество подряд идущих элементов первого массива.
- (число) `yRow`- количество подряд идущих элементов второго массива.
- (булево) `toLimitOn` - элементы чередуются до тех пор, пока пропорцию можно сохранить. Если `true` лишние элементы не включаются в результат. Если `false` добавляются в конец результата.

Пример 1 - Чередовать треки плейлистов-источников и любимые треки в соотношении 5 к 1. Удалить лишние.
```js
let tracks = Source.getTracks(playlistArray);
let savedTracks = Source.getSavedTracks();
let resultArray = Combiner.mixin(tracks, savedTracks, 5, 1, true);
```

### mixinMulti

Возвращает новый массив, в котором чередуются элементы массивов-источников в заданном соотношении. Количеством источников неограниченно.

Аргумент
- (объект) `params` - параметры, задающие источник и соотношение.

Параметры
- (массив) `source` - перечень источников
- (массив) `inRow` - перечень с количеством элементов для **каждого** источника
- (бул) `toLimitOn` - элементы чередуются до тех пор, пока соотношение можно сохранить. Если `true` лишние элементы не включаются в результат. Если `false` добавляются в конец результата, продолжая сохранять пропорцию, если это возможно. По умолчанию `false`. При `toLimitOn = true`, первая итерация проверяет количество элементов. Если элементов меньше, чем задано соотношением, вернется пустой массив.

Пример 1 - Чередовать элементы в соотношении 1:1:1. Сохранить все элементы.
```js
let x = [1, 2, 3, 4, 5];
let y = [10, 20, 30, 40];
let z = [100, 200, 300];
let result = Combiner.mixinMulti({
    source: [x, y, z],
    inRow: [1, 1, 1],
});
// 1, 10, 100, 2, 20, 200, 3, 30, 300, 4, 40, 5
```

Пример 2 - Чередовать элементы в соотношении 2:4:2 до тех пор, пока можно сохранить последовательность
```js
let x = [1, 2, 3, 4, 5];
let y = [10, 20, 30, 40];
let z = [100, 200, 300];
let result = Combiner.mixinMulti({
    toLimitOn: true,
    source: [x, y, z],
    inRow: [2, 4, 2],
});
// 1, 2, 10, 20, 30, 40, 100, 200
```

Пример 3 - Чередовать рекомендации, любимые треки и историю прослушиваний в соотношении 4:1:1 до тех пор, пока можно сохранить последовательность
```js
let recom = Source.getRecomTracks(...);
let saved = Source.getSavedTracks();
let recent = RecentTracks.get();
let tracks = Combiner.mixinMulti({
    toLimitOn: true,
    source: [recom, saved, recent],
    inRow: [4, 1, 1],
});
```

### push

Добавить в конец первого массива все элементы второго массива и так далее.

Аргументы
- (массив) `sourceArray` - первый массив, к которому добавляются элементы остальных.
- (перечень массивов) `...additionalArray` - перечень остальных массивов.

Пример 1 - Добавить элементы второго массива в конец первого массива.
```js
let firstArray = Source.getTracks(playlistArray); // допустим, 20 треков
let secondeArray = Source.getSavedTracks(); // допустим, 40 треков
Combiner.push(firstArray, secondeArray);
// теперь в firstArray 60 треков
```

Пример 2 - Добавить к первому массиву элементы двух других.
```js
let firstArray = Source.getTracks(playlistArray); // допустим, 25 треков
let secondeArray = Source.getSavedTracks(); // допустим, 100 треков
let thirdArray = Source.getPlaylistTracks(); // допустим, 20 треков
Combiner.push(firstArray, secondeArray, thirdArray);
// теперь в firstArray 145 треков
```

## Filter

Отсеивание треков по разным признакам

### dedupArtists

Удаляет дубликаты основных исполнителей по `id`. Остается только один элемент от одного исполнителя.

Аргументы
- (массив) `items` - массив треков или исполнителей, в котором требуется удалить дубликаты основных исполнителей.

Пример 1 - Удалить дубликаты основных исполнителей в треках.
```js
let tracks = Source.getTracks(playlistArray);
Filter.dedupArtists(tracks);
```

Пример 2 - Удалить дубликаты исполнителей из массива.
```js
let relatedArtists = Source.getRelatedArtists(artists);
Filter.dedupArtists(relatedArtists);
```

### dedupTracks

Удаляет дубликаты треков по `id` и `name`.

Аргументы
- (массив) `tracks` - массив треков, в котором требуется удалить дубликаты.

Пример 1 - Удалить дубликаты.
```js
let tracks = Source.getTracks(playlistArray);
Filter.dedupTracks(tracks);
```

### getDateRel

Возвращает дату со смещением в днях относительно сегодня. 

Аргументы
- (число) `days` - количество дней для смещения.
- (строка) `bound` - обнуление часов. При `startDay` 00:00, при `endDay` 23:59. Если не указать, время согласно моменту обращения.

Пример в шаблоне [любимо и забыто](/template?id=Любимо-и-забыто).

### getLastOutRange

Получить новый массив с треками, которые не прошли последнюю проверку функции [rangeTracks](/func?id=rangetracks).

Нет аргументов.

Пример 1 - Получить треки непрошедшие проверку.
```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeTracks(tracks, args);
let outRangeTracks = Filter.getLastOutRange();
```

### match

Оставляет треки, которые удовлетворяют условию `strRegex` по названию трека, альбома и основного исполнителя. В случае массива исполнителей, по их имени.

Аргументы
- (массив) `items` - массив треков или исполнителей.
- (строка) `strRegex` - строка регулярного выражения.
- (булево) `invert` - если `true` инверсия результата. По умолчанию `false`. 

Пример 1 - Удалить треки, содержащие в своем названии слова `cover` или `live`.
```js
let tracks = Source.getTracks(playlistArray);
Filter.match(tracks, 'cover|live', true);
```

### matchExcept

Оставляет только треки, которые **не** удовлетворяют условию `strRegex`.

Аргументы
- (массив) `items` - массив треков или исполнителей.
- (строка) `strRegex` - строка регулярного выражения.

Аналогично [match](/func?id=match) с аргументом `invert = true`

### matchExceptMix

Удаляет треки, содержащие `mix` и `club`.

Аргументы
- (массив) `tracks` - массив треков.

Аналогично [matchExcept](/func?id=matchexcept) с аргументом `strRegex = 'mix|club'`

### matchExceptRu

Удаляет треки, содержащие кириллицу.

Аргументы
- (массив) `tracks` - массив треков.

Аналогично [matchExcept](/func?id=matchexcept) с аргументом `strRegex = '[а-яА-ЯёЁ]+'`

### matchLatinOnly

Оставляет треки, которые содержат названия только на латинице. То есть удаляет иероглифы, кириллицу и прочее. 

Аргументы
- (массив) `tracks` - массив треков.

Аналогично [match](/func?id=match) с аргументом `strRegex = '^[a-zA-Z0-9 ]+$'`

### matchOriginalOnly

Удаляет неоригинальные версии треков.

Аргументы
- (массив) `tracks` - массив треков.

Аналогично [matchExcept](/func?id=matchexcept) с аргументом `strRegex = 'mix|club|radio|piano|acoustic|edit|live|version|cover|karaoke'`

### rangeDateAbs

Оставить только элементы, добавленные (`added_at`) или прослушанные (`played_at`) за указанный абсолютный период.

?> Для фильтра по дате релиза используйте [rangeTracks](/func?id=rangetracks).

!> Предупреждение описано в [rangeDateRel](/func?id=rangedaterel).

Аргументы
- (массив) `items` - массив элементов.
- (дата) `startDate` - стартовая граница.
- (дата) `endDate` - предельная граница.

Формат даты `YYYY-MM-DDTHH:mm:ss.sss` где
- `YYYY-MM-DD` - год, месяц, день
- `T` - разделитель для указания времени. Указать, если добавляется время.
- `HH:mm:ss.sss` - часы, минуты, секунды, миллисекунды

Пример 1 - Треки, добавленные между 1 и 3 сентября.
```js
let tracks = Source.getTracks(playlistArray);
let startDate = new Date('2020-09-01');
let endDate = new Date('2020-09-03');
Filter.rangeDateAbs(tracks, startDate, endDate);
```

Пример 2 - Треки, добавленные с 1 августа 15:00 по 20 августа 10:00.
```js
let tracks = Source.getTracks(playlistArray);
let startDate = new Date('2020-08-01T15:00');
let endDate = new Date('2020-08-20T10:00');
Filter.rangeDateAbs(tracks, startDate, endDate);
```

Пример 3 - Треки, добавленные с 1 сентября по текущую дату и время.
```js
let tracks = Source.getTracks(playlistArray);
let startDate = new Date('2020-09-01');
let endDate = new Date();
Filter.rangeDateAbs(tracks, startDate, endDate);
```

### rangeDateRel

Оставить только элементы, добавленные (`added_at`) или прослушанные (`played_at`) за указанный период относительно сегодня. 

?> Для фильтра по дате релиза используйте [rangeTracks](/func?id=rangetracks).

!> Если элемент не содержит даты, устанавливается 01.01.2000. Такое возможно, например, если трек добавлен в Spotify очень давно, источником является [getTopTracks](/func?id=gettoptracks), это плейлисты "Мой микс дня #N" или ряд других источников.

Аргументы
- (массив) `items` - массив элементов.
- (число) `sinceDays` - стартовая граница. По умолчанию сегодня 00:00.
- (число) `beforeDays` - предельная граница. По умолчанию сегодня 23:59.

Ниже пример для `sinceDays` = 7 и `beforeDays` = 2. То есть получить элементы, добавленные в плейлист с 3 сентября 00:00 по 8 сентября 23:59 относительно сегодня, 10 сентября. 

![Пример использования sinceDays и beforeDays](/img/DaysRel.png ':size=60%')

?> Механизм получения даты начала отслеживания исполнителя описан [на форуме](https://github.com/Chimildic/goofy/discussions/98)

Пример 1 - Треки, добавленные за последние 5 дней и сегодня.
```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeDateRel(tracks, 5);
// аналогично Filter.rangeDateRel(tracks, 5, 0);
```

Пример 2 - Треки за последние 7 дней исключая сегодня.
```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeDateRel(tracks, 7, 1);
```

Пример 3 - Треки за один день, который был 14 дней назад.
```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeDateRel(tracks, 14, 14);
```

Пример 4 - Треки только за сегодня.
```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeDateRel(tracks);
// аналогично Filter.rangeDateRel(tracks, 0, 0);
```

### rangeTracks

Оставляет только треки, которые удовлетворяют условиям `args`. Треки непрошедшие проверку удаляются из оригинального массива `tracks`. 

Аргументы
- (массив) `tracks` - проверяемые треки. 
- (объект) `args` - условия проверки на принадлежность диапазону `min` - `max` (границы включительно), равенству или присутствию жанра.

Категории проверки параметров
- `meta` - трек
- `features` - особенности трека
- `artist` - основной исполнитель трека
- `album` - альбом трека

!> Функция запрашивает дополнительные данные для `features`, `artist`, `album`. Чтобы сократить число запросов, используйте ее после максимального сокращения массива треков другими способами (например, [rangeDateRel](/func?id=rangedaterel), [match](/func?id=match) и другие). Полученные данные кэшируются для **текущего** выполнения. Повторный вызов функции или сортировка [sort](/func?id=sort) с теми же категориями не отправляют новых запросов.

Ниже пример объекта `args` со всеми допустимыми условиями проверки. Описание параметров читать [здесь](/guide?id=Описание-параметров-объектов).
```js
let args = {
    meta: {
        popularity: { min: 0, max: 100 },
        duration_ms: { min: 0, max: 10000 },
        explicit: false,
    },
    artist: {
        popularity: { min: 0, max: 100 },
        followers: { min: 0, max: 100000 },
        genres: ['indie'],
        ban_genres: ['rap', 'pop'],
    },
    features: {
        acousticness: { min: 0.0, max: 1.0 },
        danceability: { min: 0.0, max: 1.0 },
        energy: { min: 0.0, max: 1.0 },
        instrumentalness: { min: 0.0, max: 1.0 },
        liveness: { min: 0.0, max: 1.0 },
        loudness: { min: -60, max: 0 },
        speechiness: { min: 0.0, max: 1.0 },
        valence: { min: 0.0, max: 1.0 },
        tempo: { min: 30, max: 210 },
        key: 0,
        mode: 0,
        time_signature: 1,

        // вычисляемые https://github.com/Chimildic/goofy/discussions/87
        anger: { min: 0.0, max: 1.0 },
        happiness: { min: 0.0, max: 1.0 },
        sadness: { min: 0.0, max: 1.0 }, 
        
        // дублирует args.meta.duration_ms, достаточно одного (выбор зависит от категории)
        duration_ms: { min: 0, max: 10000 },
    },
    album: {
        popularity: { min: 30, max: 70 },
        genres: [], // Тесты показывают, что у альбомов список жанров всегда пуст
        release_date: { sinceDays: 6, beforeDays: 0 },
        // или release_date: { startDate: new Date('2020.11.30'), endDate: new Date('2020.12.30') },
    },
};
```

Пример 1 - Исключить треки жанра рэп.
```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeTracks(tracks, {
    artist: {
      ban_genres: ['rap'],
    }
});
```

Пример 2 - Оставить только треки в жанре инди и альтернативы.
```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeTracks(tracks, {
    artist: {
        genres: ['indie', 'alternative'],
    },
});
```

Пример 3 - Оставить только малопопулярные треки от малоизвестных исполнителей.
```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeTracks(tracks, {
    meta: {
      popularity: { min: 0, max: 49 },
    },
    artist: {
      followers: { min: 0, max: 9999 },
    },
});
```

### removeArtists

Удаляет из `sourceArray` исполнителей, которые есть в `removedArray`. Совпадение определяется по `id` основного исполнителя трека.

Аргументы
- (массив) `sourceArray` - массив треков или исполнителей, в котором нужно удалить.
- (массив) `removedArray` - массив треков или исполнителей, которые требуется удалить.
- (бул) `invert` - инверсия результата. Если `true`, удалять все, кроме того, что в `removedArray`. По умолчанию `false`.

Пример 1 - Получить треки плейлистов и исключить исполнителей любимых треков.
```js
let sourceArray = Source.getTracks(playlistArray);
let removedArray = Source.getSavedTracks();
Filter.removeArtists(sourceArray, removedArray);
```

### removeTracks

Удаляет из `sourceArray` треки, которые есть в `removedArray`. Совпадение определяется по `id` трека или по названию трека вместе с исполнителем.

Аргументы
- (массив) `sourceArray` - массив треков, в котором нужно удалить треки.
- (массив) `removedArray` - массив треков, которые требуется удалить.
- (бул) `invert` - инверсия результата. Если `true`, удалять все треки, кроме тех, что в `removedArray`. По умолчанию `false`.

?> Возможно будет полезным не удалить, а заменить треки на новые с помощью [replaceWithSimilar](/func?id=replacewithsimilar)

!> Существует вероятность столкнуться с [проблемой релинка](https://github.com/Chimildic/goofy/discussions/99) при удалении истории прослушиваний

Пример 1 - Получить треки плейлистов и исключить любимые треки.
```js
let sourceArray = Source.getTracks(playlistArray);
let removedArray = Source.getSavedTracks();
Filter.removeTracks(sourceArray, removedArray);
```

### removeUnavailable

Удаляет треки, которые нельзя послушать. Изменяет содержание оригинального массива. Не заменяет оригинал на аналог ([релинк](https://developer.spotify.com/documentation/general/guides/track-relinking-guide/), т.е. нет переадресации на другой похожий трек).

?> Совершает дополнительные запросы (1 на 50 треков). В случае, если трек находится в неопределенном состоянии.

?> Допустимо применять фильтр для треков из `Cache`, прошедших сжатие [compressTracks](/func?id=compresstracks). Если сжатия не было, состояние определяется по значению в кэшированном треке.

Аргументы
- (массив) `tracks` - треки, которые нужно фильтровать.
- (строка) `market` - страна, в которой проверяется доступность треков. По умолчанию `RU`.

Пример 1 - Удалить недоступные в России треки плейлиста
```js
let tracks = Source.getPlaylistTracks('', 'id');
Filter.removeUnavailable(tracks);
```

### replaceWithSimilar

Заменяет треки на похожие. На одну замену один случайный трек из результатов [getRecomTracks](/func?id=getrecomtracks). Если замены нет трек удаляется.

Аргументы
- (массив) `originTracks` - где заменять
- (массив) `replacementTracks` - что заменять (можно несколько)

Пример 1 - Заменить недавно игравшие треки и лайки плейлиста на близкие аналоги
```js
let tracks = Source.getPlaylistTracks('', 'id');
Filter.replaceWithSimilar(tracks, RecentTracks.get(2000), Source.getSavedTracks());
```

## Lastfm

Модуль по работе с сервисом Last fm

### convertToSpotify

Возвращает массив по `type`, элементы которого найдены поиском Spotify.

Аргументы
- (массив) `items` - элементы в lastfm-формате. Например, полученные из [getCustomTop](/func?id=getcustomtop) при `isRawItems = true`.
- (строка) `type` - тип поиска: `track`, `artist` или `album`. По умолчанию `track`.

[Пример в шаблоне](/template?id=Исполнители-одного-хита)

### getCustomTop

Возвращает массив элементов по `type`, отсортированных по количеству прослушиваний за указанный период. К объектам результата добавится ключ `countPlayed` со значением числа прослушиваний

Аргумент
- (объект) `params` - перечень параметров для отбора 

Описание параметров
- (строка) `user` - логин пользователя Lastfm, чей топ собирать
- (дата/строка/число) `from` - дата старта.
- (дата/строка/число) `to` - дата окончания.
- (строка) `type` - вариация результата: `track`, `artist` или `album`. По умолчанию `track`.
- (число) `count` - количество элементов. По умолчанию 40. Только это количество треков будет искаться в Spotify.
- (число) `offset` - пропуск первых N элементов. По умолчанию 0.
- (число) `minPlayed` - минимальное количество прослушиваний, включительно. По умолчанию 0.
- (число) `maxPlayed` - максимальное количество прослушиваний, включительно. По умолчанию 100 тысяч.
- (бул) `isRawItems` - если `true`, результат из lastfm-элементов. Игнорируются `count` и `offset`. Когда не указано или `false`, поиск элементом по названию в Spotify.

?> На форуме есть [подробный пример](https://github.com/Chimildic/goofy/discussions/91) использования, позволяющий получить _рекомендации из прошлого_

Пример 1 - Получить топ 40 треков за 2015 год
```js
let topTracks = Lastfm.getCustomTop({
    user: 'login',
    from: '2015-01-01', // или new Date('2015-01-01'),
    to: '2015-12-31', // или new Date('2015-12-31').getTime(),
});
```

Пример 2 - Получить топ 10 исполнителей за первое полугодие 2014 года
```js
let topArtists = Lastfm.getCustomTop({
    user: 'login',
    type: 'artist',
    from: '2014-01-01',
    to: '2014-06-30',
    count: 10,
});
```

### getLibraryStation

Возвращает массив треков из радио last fm `Библиотека`. Содержит только заскроббленные ранее треки. Внимание на предупреждение из [getRecentTracks](/func?id=getrecenttracks).

Аргументы
- (строка) `user` - логин пользователя, чье радио является источником.
- (число) `countRequest` - количество запросов к last fm. Один запрос дает примерно от 20 до 30 треков.

Пример использования
```js
let tracks = Lastfm.getLibraryStation('login', 2);
```

### getLovedTracks

Возвращает массив любимых треков пользователя `user`, ограниченного количеством `limit`. Внимание на предупреждение из [getRecentTracks](/func?id=getrecenttracks). Включает дату добавления, можно использовать фильтр по дате.

Аргументы
- (строка) `user` - логин пользователя Last.fm, чьи любимые треки нужно искать.
- (число) `limit` - предельное количество треков.

Пример 1 - Получить 200 любимых треков
```js
let tracks = Lastfm.getLovedTracks('login', 200);
```

### getMixStation

Возвращает массив треков из радио last fm `Микс`. Содержит ранее заскроббленные треки и рекомендации last fm. Внимание на предупреждение из [getRecentTracks](/func?id=getrecenttracks).

Аргументы
- (строка) `user` - логин пользователя, чье радио является источником.
- (число) `countRequest` - количество запросов к last fm. Один запрос дает примерно от 20 до 30 треков.

Пример использования
```js
let tracks = Lastfm.getMixStation('login', 2);
```

### getNeighboursStation

Возвращает массив треков из радио last fm `Соседи`. Содержит треки, которые слушают пользователи last fm со схожими вам музыкальными вкусами. Внимание на предупреждение из [getRecentTracks](/func?id=getrecenttracks).

Аргументы
- (строка) `user` - логин пользователя, чье радио является источником.
- (число) `countRequest` - количество запросов к last fm. Один запрос дает примерно от 20 до 30 треков.

Пример использования
```js
let tracks = Lastfm.getNeighboursStation('login', 2);
```

### getRecentTracks

Возвращает массив недавно прослушанных треков пользователя `user`, ограниченного количеством `limit`. 

!> Источником треков является lastfm. Эквивалент трека находится поиском Spotify по наилучшему совпадению. Если совпадения нет, трек игнорируется. Один трек lastfm равен одному запросу поиска. Будьте внимательны с [ограничениями](/overview?id=Ограничения) по количеству запросов в день и времени выполнения.

Аргументы
- (строка) `user` - логин пользователя Last.fm, чью историю прослушиваний нужно искать.
- (число) `count` - предельное количество треков.

Пример 1 - Получить 200 недавно прослушанных треков
```js
let tracks = Lastfm.getRecentTracks('login', 200);
```

### getRecomStation

Возвращает массив треков из радио last fm `Рекомендации`. Содержит только рекомендации last fm. Внимание на предупреждение из [getRecentTracks](/func?id=getrecenttracks).

Аргументы
- (строка) `user` - логин пользователя, чье радио является источником.
- (число) `countRequest` - количество запросов к last fm. Один запрос дает примерно от 20 до 30 треков.

Пример использования
```js
let tracks = Lastfm.getRecomStation('login', 2);
```

### getSimilarArtists

Возвращает массив исполнителей, которые похожи на входные элементы согласно данным Lastfm.

Аргументы
- (массив) `items` - массив треков или исполнителей. Выбирается только `name` исполнителя.
- (число) `match` - минимальное значение похожести на оригинального исполнителя в границе от `0.0` до `1.0`. 
- (число) `limit` - количество запрашиваемых похожих исполнителей на одного оригинального.
- (бул) `isFlat` - если `false` результат содержит исполнителей в отдельном массиве. Если `true` все исполнители в одном массиве. По умолчанию `true`.

Пример 1 - Получить исполнителей похожих на отслеживаемых
```js
let artists = Source.getArtists({ followed_include: true, });
let similarArtists = Lastfm.getSimilarArtists(artists, 0.65, 20);
```

### getSimilarTracks

Возвращает массив треков, которые похожи на входные треки согласно данным Lastfm.

Аргументы
- (массив) `tracks` - треки, для которых нужно найти похожие.
- (число) `match` - минимальное значение похожести на оригинальный трек в границе от `0.0` до `1.0`. 
- (число) `limit` - количество запрашиваемых похожих треков на один оригинальный трек.
- (бул) `isFlat` - если `false` результат содержит треки в отдельном массиве. Если `true` все треки в одном массиве. По умолчанию `true`.

Пример 1 - Получить треки, похожие на плейлист
```js
let playlistTracks = Source.getPlaylistTracks('name', 'id');
let similarTracks = Lastfm.getSimilarTracks(playlistTracks, 0.65, 30);
```

### getTopAlbums

Возвращает массив с топом альбомов по заданному периоду.

Аргументы
- (объект) `params` - параметры для выбора топа альбома. Аналогично параметрам [getTopTracks](/func?id=gettoptracks-1).

Пример 1 - Получить топ-10 альбомов за полгода
```js
let artists = Lastfm.getTopAlbums({
  user: 'ваш логин',
  period: '6month',
  limit: 10
});
```

### getTopArtists

Возвращает массив с топом исполнителей по заданному периоду.

Аргументы
- (объект) `params` - параметры для выбора топа исполнителей. Аналогично параметрам [getTopTracks](/func?id=gettoptracks-1).

Пример 1 - Получить топ-10 исполнителей за полгода
```js
let artists = Lastfm.getTopArtists({
  user: 'ваш логин',
  period: '6month',
  limit: 10
});
```

### getTopTracks

Возвращает массив с топом треков по заданному периоду. Внимание на предупреждение из [getRecentTracks](/func?id=getrecenttracks).

Аргументы
- (объект) `params` - параметры для выбора топа треков.

Допустимые значения `params`
```js
{
  user: 'login', // логин пользователя last.fm
  period: 'overall', // период, допустимо: overall | 7day | 1month | 3month | 6month | 12month
  limit: 50 // предельное количество треков
}
```

Пример 1 - Получить топ-40 треков за полгода
```js
let tracks = Lastfm.getTopTracks({
  user: 'ваш логин',
  period: '6month',
  limit: 40
});
```

### removeRecentArtists
Удаляет из массива треков `sourceArray` историю недавно прослушанных `limit` треков пользователя `lastfmUser`. Совпадение определяется только по имени исполнителя. Требуется [дополнительная настройка](/install?id=Настройка-lastfm).

Аргументы
- (массив) `sourceArray` - массив треков, в котором нужно удалить треки.
- (строка) `user` - логин пользователя Last.fm, чью историю прослушиваний нужно исключить.
- (число) `count` - предельное количество треков истории прослушиваний. По умолчанию 600.

Пример как у [removeRecentTracks](/func?id=removerecenttracks)

### removeRecentTracks

Удаляет из массива треков `sourceArray` историю недавно прослушанных `limit` треков пользователя `lastfmUser`. Совпадение определяется по названию трека и исполнителя. Требуется [дополнительная настройка](/install?id=Настройка-lastfm).

Аргументы
- (массив) `sourceArray` - массив треков, в котором нужно удалить треки.
- (строка) `user` - логин пользователя Last.fm, чью историю прослушиваний нужно исключить.
- (число) `count` - предельное количество треков истории прослушиваний. По умолчанию 600.

Пример 1 - Создать плейлист с любимыми треками, которые не были прослушаны за последние 5 тысяч скробблов Last.fm пользователя `login`
```js
let savedTracks = Source.getSavedTracks();
Lastfm.removeRecentTracks(savedTracks, 'login', 5000)
Playlist.saveAsNew({
  name: 'Давно не слушал',
  tracks: savedTracks,
});
```

## Library

Действия над любимыми треками и подписками на исполнителей

### deleteAlbums

Удалить альбомы из библиотеки.

Аргументы
- (массив) `albums` - перечень альбомов для удаления. Значимо только `id`.

### deleteFavoriteTracks

Удалить треки из любимых (снять лайки)

Аргументы
- (массив) `tracks` - перечень треков. Значимо только `id`.

Пример 1 - Очистить все лайки Spotify
```js
let savedTracks = Source.getSavedTracks();
Library.deleteFavoriteTracks(savedTracks);
```

### followArtists

Подписаться на исполнителей

Аргументы
- (массив) `artists` - перечень исполнителей. Значимо только `id`.

Пример в [Yandex.getArtists](/func?id=getartists)

### saveAlbums

Добавить альбомы в библиотеку.

Аргументы
- (массив) `albums` - перечень альбомов для добавления. Значимо только `id`.

### saveFavoriteTracks

Добавить треки в любимые (поставить лайк)

Аргументы
- (массив) `tracks` - перечень треков. Значимо только `id`.

Пример 1 - Добавить последние 50 лайков из Яндекс в Spotify
```js
let yandexTracks = Yandex.getTracks('owner', '3', 50);
let savedTracks = Source.getSavedTracks();
Filter.removeTracks(yandexTracks, savedTracks);
Library.saveFavoriteTracks(yandexTracks);
```

### unfollowArtists

Отписаться от исполнителей

Аргументы
- (массив) `artists` - перечень исполнителей. Значимо только `id`.

Пример аналогичен [Yandex.getArtists](/func?id=getartists). Только использовать `unfollowArtists`.

## Order

Сортировка треков

### reverse

Обратная сортировка. Первый элемент станет последним и наоборот.

Аргументы
- (массив) `array` - массив, чьи элементы необходимо отсортировать в обратном направлении.

Пример 1 - Обратная сортировка
```js
let array = [1, 2, 3, 4, 5, 6];

Order.reverse(array);
// результат 6, 5, 4, 3, 2, 1

Order.reverse(array);
// результат 1, 2, 3, 4, 5, 6
```

Пример 2 - Обратная сортировка треков плейлиста
```js
let tracks = Source.getTracks(playlistArray);
Order.reverse(tracks);
```

### separateArtists

Сортировка, при которой соблюдается минимальный отступ между одним и тем же исполнителем. Треки, которые не удалось разместить будут исключены.

Аргументы
- (массив) `tracks` - массив треков, который нужно отсортировать.
- (число) `space` - значение минимального отступа.
- (булево) `isRandom` - влияет на сортировку. Если `true` выполняется случайная сортировка оригинального массива, что повлияет на порядок при разделении исполнителей. Если `false` без случайной сортировки. Тогда результат при одинаковых входных треках будет тоже одинаковыми. По умолчанию `false`.

Пример 1 - Условный пример разделения
```js
let array = ['cat', 'cat', 'dog', 'lion']
Order.separateArtists(array, 1, false);
// результат cat, dog, cat, lion

array = ['cat', 'cat', 'dog', 'lion']
Order.separateArtists(array, 1, false);
// повторный вызов, результат тот же: cat, dog, cat, lion

array = ['cat', 'cat', 'dog', 'lion']
Order.separateArtists(array, 1, true);
// повторный вызов и случайная сортировка: cat, lion, dog, cat
```

Пример 2 - Разделить одного и того же исполнителя минимум двумя другими.
```js
let tracks = Source.getTracks(playlistArray);
Order.separateArtists(tracks, 2);
```

### separateYears

Возвращает объект, в котором по ключу с годом будет массив с треками, вышедших в этот год. Треки массива не сортируются.

Аргументы
- (массив) `tracks` - массив треков, которые нужно разделить по годам.

Пример 1 - Получить треки вышедшие только в 2020 году
```js
let tracks2020 = Order.separateYears(tracks)['2020'];
```

Пример 2 - Возможна ошибка, при которой среди треков нет указанного года. Выберите одно из:
- Используйте [pickYear](/func?id=pickyear)
- Подмените пустым массивом
- Проверьте условием
  
```js
// Подменить на пустой массив, если нет треков указанного года
let tracks2020 = Order.separateYears(tracks)['2020'] || [];

// Проверка через условие
let tracksByYear = Order.separateYears(tracks);
if (typeof tracksByYear['2020'] != 'undefined'){
    // треки есть
} else {
    // треков нет
}
```

### shuffle

Перемешивает элементы массива случайным образом.

Аргументы
- (массив) `array` - массив, чьи элементы необходимо перемешать.

Пример 1 - Случайное перемешивание
```js
let array = [1, 2, 3, 4, 5, 6];

Order.shuffle(array);
// результат 3, 5, 4, 6, 2, 1

Order.shuffle(array);
// результат 6, 1, 2, 3, 5, 4

Order.shuffle(array);
// результат 6, 5, 2, 3, 1, 4
```

Пример 2 - Перемешать треки
```js
let tracks = Source.getTracks(playlistArray);
Order.shuffle(tracks);
```

### sort

Сортирует оригинальный массив по заданному ключу.

!> Функция делает дополнительные запросы. Чтобы сократить число запросов, используйте ее после максимального сокращения массива треков другими способами. Подробнее в [rangeTracks](/func?id=rangetracks).

Аргументы
- (массив) `source` - массив, который нужно отсортировать. Допустим массив треков, исполнителей или альбомов.
- (строка) `pathKey` - ключ сортировки.
- (строка) `direction` - направление сортировки: `asc` по возрастанию, `desc` по убыванию. По умолчанию `asc`.

Допустимые ключи в формате `категория.ключ`. Описание ключей [здесь](/guide?id=Описание-параметров-объектов). При использовании категории `features` общий поток любых треков становится приятнее.

| Категория | Ключ |
|-|-|
| meta | name, popularity, duration_ms, explicit, added_at, played_at |
| features | acousticness, danceability, energy, instrumentalness, liveness, loudness, speechiness, valence, tempo, key, mode, time_signature, duration_ms |
| artist | popularity, followers, name |
| album | popularity, name, release_date |

?> Если смешивается несколько источников треков, не у всех есть указанный ключ. Например, `played_at` есть только у истории прослушиваний.

Пример 1 - Сортировка по убывающей популярности исполнителей
```js
Order.sort(tracks, 'artist.popularity', 'desc');
```

Пример 2 - Сортировка по возрастающей энергичности
```js
Order.sort(tracks, 'features.energy', 'asc');
```

## Playlist

Создание или обновление плейлиста

### getDescription

Возвращает строку вида: `Исполнитель 1, Исполнитель 2... и не только`.

Аргументы
- (массив) `tracks` - треки, из которых случайно выбираются исполнители.
- (число) `limit` - количество случайно выбираемых исполнителей. По умолчанию 5.

Пример 1 - Создать плейлист с описанием
```js
let tracks = Source.getTracks(playlistArray);
Playlist.saveWithReplace({
    id: 'abcd',
    name: 'Большой микс дня',
    tracks: tracks,
    description: Playlist.getDescription(tracks),
});
```

### removeTracks

Удаляет треки из плейлиста.

Аргументы
- (строка) `id` - id плейлиста.
- (массив) `tracks` - массив треков.

Пример 1 - Удалить лайки из плейлиста
```js
let savedTracks = Source.getSavedTracks();
Playlist.removeTracks('id', savedTracks);
```

### saveAsNew

Создает плейлист. Каждый раз новый.

Аргументы
- (объект) `data` - данные для создания плейлиста.

Формат данных для создания плейлиста
- (строка) `name` - название плейлиста, обязательно.
- (массив) `tracks` - массив треков, обязательно.
- (строка) `description` - описание плейлиста. До 300 символов.
- (булево) `public` - если `false` плейлист будет приватным. По умолчанию `true`.
- (строка) `sourceCover` - прямая ссылка на обложку (до 256 кб). Если указано, `randomCover` игнорируется. 
- (строка) `randomCover` - добавить случайную обложку при значении `once`. Без использования, стандартная мозайка от Spotify.

Пример 1 - Создать публичный плейлист с любимыми треками без описания со случайной обложкой
```js
let tracks = Source.getSavedTracks();
Playlist.saveAsNew({
  name: 'Копия любимых треков',
  tracks: tracks,
  randomCover: 'once',
  // sourceCover: tracks[0].album.images[0].url,
});
```

Пример 2 - Создать приватный плейлист с недавней историей прослушиваний и описанием без обложки.
```js
let tracks = RecentTracks.get(200);
Playlist.saveAsNew({
  name: 'История прослушиваний',
  description: '200 недавно прослушанных треков'
  public: false,
  tracks: tracks,
});
```

### saveWithAppend

Добавляет треки к уже имеющимся в плейлисте. Обновляет остальные данные (название, описание). Если плейлиста еще нет, создает новый.

Аргументы
- (объект) `data` - данные о плейлисте. Формат данных о плейлисте согласно описанию [saveWithReplace](/func?id=savewithreplace).
- (булево) `toEnd` - если `true`, добавляет треки в конец списка. Если `false`, в начало. По умолчанию `false`.

Пример 1 - Добавить треки в начало плейлиста.
```js
let tracks = Source.getTracks(playlistArray);
Playlist.saveWithAppend({
    id: 'fewf4t34tfwf4',
    name: 'Микс дня',
    tracks: tracks
});
```

Пример 2 - Добавить треки в конец плейлиста, обновить название и описание.
```js
let tracks = Source.getTracks(playlistArray);
Playlist.saveWithAppend({
    id: 'fewf4t34tfwf4',
    name: 'Новое название',
    description: 'Новое описание',
    tracks: tracks,
    toEnd: true,
});
```

!> Если обновить название плейлиста без указания `id` будет создан новый плейлист. Потому что поиск не найден плейлист с новым названием.

### saveWithReplace

Заменяет треки плейлиста. Обновляет остальные данные (название, описание). Если плейлиста еще нет, создает новый.

Аргументы
- (объект) `data` - данные о плейлисте.

Формат данных о плейлисте
- (строка) `id` - [идентификационный номер плейлиста](#идентификатор).
- (строка) `name` - название плейлиста, обязательно.
- (массив) `tracks` - массив треков, обязательно.
- (строка) `description` - описание плейлиста. До 300 символов.
- (булево) `public` - если `false` плейлист будет приватным. По умолчанию `true`.
- (строка) `sourceCover` - прямая ссылка на обложку (до 256 кб). Если указано, `randomCover` игнорируется. 
- (строка) `randomCover` - если `once` добавит случайную обложку. При `update` каждый раз обновляет обложку. Без использования, стандартная мозайка от Spotify.

?> Рекомендуется всегда указывать `id`. Если `id` не указано, поиск по названию. Если такого плейлиста нет, создается новый.

Пример 1 - Обновить содержимое плейлиста и обложку
```js
let tracks = Source.getTracks(playlistArray);
Playlist.saveWithReplace({
    id: 'fewf4t34tfwf4',
    name: 'Микс дня',
    description: 'Описание плейлиста',
    tracks: tracks,
    randomCover: 'update',
    // sourceCover: tracks[0].album.images[0].url,
});
```

Пример 2 - Обновить содержимое плейлиста из примера 1. Поиск по названию.
```js
let tracks = RecentTracks.get();
Playlist.saveWithReplace({
    name: 'История',
    description: 'Новое описание плейлиста',
    tracks: tracks,
    randomCover: 'update',
});
```

### saveWithUpdate

Обновляет треки плейлиста: добавляет новые; удаляет те, которых нет в массиве; сохраняет оригинальную дату добавления треков. Теряется сортировка заданная в массиве. Игнорируются дубликаты по `id`.

Аргументы
- (объект) `data` - данные о плейлисте, соответствует [saveWithReplace](/func?id=savewithreplace).

Дополнительный режим для `data`
- (булево) `toEnd` - если `true`, добавляет треки в конец списка. Если `false`, в начало. По умолчанию `false`.

## RecentTracks

Источник истории прослушиваний 

### appendTracks

Добавляет массив треков к файлу истории прослушиваний. Дата добавления в плейлист `added_at` становится датой прослушивания `played_at`. Если даты нет, устанавливается значение `01.01.2000`. Сортируется по дате прослушивания от свежих к более старым.

?> Если дата добавляемого трека из плейлиста новее даты последнего прослушивания, добавляемый трек станет первым в списке. Это вызовет логическую ошибку вычисления последнего прослушивания в триггере. Поэтому вместо игнорирования или пары треков, триггер добавит весь доступный массив. По умолчанию это 50 треков для Spotify, 30 для Lastfm. Следующий триггер сработает корректно. 

!> Обратите внимание на ограничение в 20 тысяч треков для истории прослушиваний. Предел можно увеличить.

Аргументы
- (строка) `filename` - имя файла истории прослушиваний. Допустимо: `SpotifyRecentTracks` и `LastfmRecentTracks`.
- (массив) `tracks` - добавляемые треки.

Пример 1 - Добавить в историю прослушиваний все любимые треки
```js
let tracks = Source.getSavedTracks();
RecentTracks.appendTracks('SpotifyRecentTracks', tracks);
```

### compress

Сжимает треки в существующих накопительных файлах истории прослушиваний в зависимости от [параметров](/guide?id=Параметры). Предварительно создает копию файла.

Аргументов нет. Возвращаемого значения нет.

?> Используется для совместимости с прошлыми версиями библиотеки. Достаточно одного выполнения, чтобы сжать файлы истории прослушиваний. Новые треки истории сжимаются автоматически.

Пример 1 - Сжать существующие файлы с историей прослушиваний. Файлы выбираются исходя из [активных параметров](/guide?id=Параметры).
```js
RecentTracks.compress();
```

### get

Возвращает массив треков истории прослушиваний в зависимости от [параметров](/guide?id=Параметры). Сортировка по дате прослушивания от свежих к более старым. 

Аргументы
- (число) `limit` - если указано, ограничить количество возвращаемых треков. Если нет, все доступные. 

| Включенный параметр | Возвращаемый массив |
|-|-|
| `ON_SPOTIFY_RECENT_TRACKS` | История прослушиваний только Spotify |
| `ON_LASTFM_RECENT_TRACKS` | История прослушиваний только Lastfm.   |
| `ON_SPOTIFY_RECENT_TRACKS` и `ON_LASTFM_RECENT_TRACKS` | Объединение обоих источников с удалением дубликатов треков. |

Пример 1 - Получить массив треков истории прослушиваний. Источник треков зависит от параметров. 
```js
let tracks = RecentTracks.get();
```

Пример 2 - Получить 100 треков истории прослушиваний.
```js
let tracks = RecentTracks.get(100);
```

### getPlayback

Возвращает объект, содержащий играющий трек и сопутствующий контекст плеера. Пример объекта [здесь](https://developer.spotify.com/documentation/web-api/reference/#endpoint-get-information-about-the-users-current-playback). Если данных нет, объект пустой.

Аргументов нет.

[Пример использования](https://github.com/Chimildic/goofy/discussions/102)

## Search

Модуль поиска Spotify по ключевым словам. Используется другими функциями. Понадобится только для индивидуальных решений.

### find*

Возвращает массив с результатами поиска по ключевому слову для типов: плейлист, трек, альбом, исполнитель.

Аргументы
- (массив) `keywords` - перечень ключевых слов, только строки. На каждое в результатах будет свой массив.
- (число) `requestCount` - количество запросов на **одно** ключевое слово. На один запрос 50 объектов, если они есть. Максимум 40 запросов. По умолчанию 1.

Пример 1 - Найти 100 плейлистов по слову `rain`

?> Лучшим способом будет функция [mineTracks](/func?id=minetracks). Прямое использование модуля Search нужно для решений, которые не реализованы по умолчанию. Например, при [импорте треков с FM-радио](https://github.com/Chimildic/goofy/discussions/35).

```js
let keywords = ['rain'];
let playlists = Search.findPlaylists(keywords, 2);
```

### getNoFound

Возвращает массив с ключевыми словами и типом поиска, для которых не нашлось результатов за текущее выполнение скрипта. 

Аргументов нет.

```js
let noFound = Search.getNoFound();
// структура: { type: '', keyword: '', item: {} }
```

### multisearch*

Возвращается лучшее совпадение по ключевому словаму трека/исполнителя/альбома

Аргументы
- (массив) `items` - перебираемые элементы
- (функция) `parseNameMethod` - коллбэк, вызываемый для каждого элемента. Должен вернуть строку, являющуюся ключевым словом для поиска.

Пример 1 - Когда массив элементов с простым текстом
```js
let keywords = ['skillet', 'skydive'];
let artists = Search.multisearchArtists(keywords, (i) => i);
```

Пример 2 - Когда массив элементов со сложной структурой
```js
let tracks = Search.multisearchTracks(items, (item) => {
    return `${item.artist} ${item.title}`.formatName();
});
```

### sendMusicRequest

Заполняет форму [music request](https://docs.google.com/forms/d/e/1FAIpQLScMGwTBnCz8nOPkM5g9IwwbpKolEWOXkhpAUSl8JjlkKcBGKw/viewform) данными из [getNoFound](/func?id=getnofound). То есть отправляет запрос на добавление исполнителей или треков, которых не удалось найти в Spotify.

?> Эксперимент. Используйте после запросов к lastfm, например `getCustomTop`, в конце функции.

Аргумент
- (объект) `context` - контекст текущего массива ненайденных элементов для сохранения в кэш файл `NoFoundItems`. Позволит выполнить повторный поиск позже, а также узнать куда планировалось добавлять элемент.

Пример 1 - Выполнить некоторый поиск треков в lastfm и отправить запрос на добавления для ненайденных
```js
let tracks = Lastfm.getCustomTop({
  user: KeyValue.LASTFM_LOGIN,
  from: '2013-01-01',
  to: '2013-12-12',
});
Search.sendMusicRequest({ name: 'Топ 2013', id: 'abc' });
```

Пример 2 - Повторный поиск треков, который планировали добавить в плейлист с `id = abc` 
```js
let file = Cache.read('NoFoundItems');
let itemTracks = file.filter(i => i.context.id == 'abc').map(i => i.items).flat(1);
let tracks = Search.multisearchTracks(itemTracks, (item) => item.keyword);
```

## Selector

Отбор количества треков по позиции

### isDayOfWeek

Возвращает булево значение: `true` если сегодня день недели `strDay` и `false` если нет.

Аргументы
- (строка) `strDay` - день недели.
- (строка) `locale` - локаль дня недели. По умолчанию `en-US`, для которой допустимы значения: `sunday`, `monday`, `tuesday`, `wednesday`, `thursday`, `friday`, `saturday`.

Пример использования
```js
if (Selector.isDayOfWeek('friday')){
    // сегодня пятница
} else {
    // другой день недели
}
```

### isDayOfWeekRu

Возвращает булево значение: `true` если сегодня день недели `strDay` и `false` если нет. Значение дня недели кириллицей.

Аргументы
- (строка) `strDay` - день недели. Допустимые значения: `понедельник`, `вторник`, `среда`, `четверг`, `пятница`, `суббота`, `воскресенье`.

Пример использования
```js
if (Selector.isDayOfWeekRu('понедельник')){
    // сегодня понедельник
} else if (Selector.isDayOfWeekRu('среда')) {
    // сегодня среда
} else {
    // другой день недели
}
```

### isWeekend

Возвращает булево значение: `true` если сегодня суббота или пятница и `false` если нет.

Аргументов нет.

Пример использования
```js
if (Selector.isWeekend()){
    // сегодня выходной
} else {
   // будни
}
```

### keepAllExceptFirst / sliceAllExceptFirst

Изменяет / возвращает массив, состоящий из всех элементов массива `array` кроме `skipCount` первых.

> Разница функций `keep*` и `slice*`:
> 
> - `keep*` изменяет содержимое оригинального массива, 
> - `slice*` возвращает новый массив, не изменяя оригинала.

Аргументы
- (массив) `array` - массив, из которого берутся элементы.
- (число) `skipCount` - количество пропускаемых элементов.

Пример 1 - Получить все треки кроме первых 10.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceAllExceptFirst(tracks, 10);
```

### keepAllExceptLast / sliceAllExceptLast

Изменяет / возвращает массив, состоящий из всех элементов массива `array` кроме `skipCount` последних.

Аргументы
- (массив) `array` - массив, из которого берутся элементы.
- (число) `skipCount` - количество пропускаемых элементов.

Пример 1 - Получить все треки кроме последних 10.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceAllExceptLast(tracks, 10);
```

### keepFirst / sliceFirst

Изменяет / возвращает массив, состоящий из первых `count` элементов массива `array`.

Аргументы
- (массив) `array` - массив, из которого берутся элементы.
- (число) `count` - количество элементов.

Пример 1 - Получить первые 100 треков.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceFirst(tracks, 100);
```

### keepLast / sliceLast

Изменяет / возвращает массив, состоящий из последних `count` элементов массива `array`.

Аргументы
- (массив) `array` - массив, из которого берутся элементы.
- (число) `count` - количество элементов.

Пример 1 - Получить последние 100 треков.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceLast(tracks, 100);
```

### keepNoLongerThan / sliceNoLongerThan

Изменяет / возвращает массив треков с общей длительностью не более, чем `minutes` минут.

Аргументы
- (массив) `tracks` - исходный массив треков.
- (число) `minutes` - количество минут.

Пример 1 - Получить треки с общей продолжительностью не более, чем 60 минут.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceNoLongerThan(tracks, 60);
```

### keepRandom / sliceRandom

Изменяет / возвращает массив, состоящий из случайно отобранных элементов исходного массива.

Аргументы
- (массив) `array` - массив, из которого берутся элементы.
- (число) `count` - количество случайно выбираемых элементов.

Пример 1 - Получить 20 случайных треков.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceRandom(tracks, 20);
```

### pickYear

Возвращает массив треков, релиз которых был в указанном году. Если таких треков нет, выбирается ближайший год.

Аргументы
- (массив) `tracks` - треки, среди которых выбирать.
- (строка) `year` - год релиза.
- (число) `offset` - допустимое смещение для ближайшего года. По умолчанию 5.

Пример 1 - Выбрать любимые треки, вышедшие в 2020 году
```js
let tracks = Selector.pickYear(savedTracks, '2020');
```

### sliceCopy

Возвращает новый массив, который является копией исходного массива.

?> Используйте создание копии, если в одном скрипте нужно выполнить разные действия над источником. Позволит ускорить время выполнения и не отправлять тех же запросов дважды.

Аргументы
- (массив) `array` - исходный массив, копию которого нужно создать.

Пример 1 - Создать копию массива.
```js
let tracks = Source.getTracks(playlistArray);
let tracksCopy = Selector.sliceCopy(tracks);
```

## Source

Источник получения треков Spotify

### craftTracks

Возвращает массив треков, полученный от [getRecomTracks](/func?id=getrecomtracks) для каждой пятерки элементов исходных треков. Дубликаты исходных треков игнорируются, в рекомендованных удаляются. Ограничение в пять элементов продиктовано Spotify API для функции рекомендаций. Можно частично повлиять на формируемые пятерки элементов. До вызова функции применив одну из сортировок `Order`.

Аргументы
- (массив) `tracks` - треки для которых получать рекомендации. При `key` равному `seed_artists` допустим массив исполнителей.
- (объект) `params` - дополнительные параметры.

Описание параметров
- (строка) `key` - определяет по какому ключу рекомендации. Допустимо: `seed_tracks` и `seed_artists`. По умолчанию `seed_tracks`.
- (объект) `query` - необязательный параметр, доступны все ключи [getRecomTracks](/func?id=getrecomtracks), кроме указанного в `key`.

?> В `query` можно указать два из: `seed_tracks`, `seed_artists`, `seed_genres`. Третий выбирается исходя из `key`. Таким образом, можно задать статичные трек/исполнителя/жанр (до 4 значений на все). Оставшиеся свободные места будут подставляться исходя из `key`.

Пример 1 - Получить рекомендации по всем любимым трекам по их исполнителям
```js
let tracks = Source.getSavedTracks();
let recomTracks = Source.craftTracks(tracks, {
    key: 'seed_artists',
    query: {
        limit: 20, // по умолчанию и максимум 100
        min_energy: 0.4,
        min_popularity: 60,
        // target_popularity: 60,
    }
});
```

Пример 2 - Рекомендации с указанием статичного жанра и трека. Оставшиеся 3 места занимаются `seed_artists`.
```js
let recomTracks = Source.craftTracks(tracks, {
    key: 'seed_artists',
    query: {
        seed_genres: 'indie',
        seed_tracks: '6FZDfxM3a3UCqtzo5pxSLZ'
    }
});
```

Пример 3 - Можно указать только массив треков. Тогда будут рекомендации по ключу `seed_tracks`.
```js
let tracks = Source.getSavedTracks();
let recomTracks = Source.craftTracks(tracks);
```

### getAlbumsTracks

Возвращает массив треков из всех альбомов.

Аргументы
- (массив) `albums` - перечень альбомов

Пример 1 - Получить треки из топ-10 альбомов Lastfm
```js
let albums = Lastfm.getTopAlbums({ user: 'login', limit: 10 });
let tracks = Source.getAlbumsTracks(albums);
```

### getAlbumTracks

Возвращает массив треков указанного альбома.

Аргументы
- (объект) `album` - объект одного альбома
- (число) `limit` - если указано, выбирает треки случайно до указанного количества.

Пример 1 - Получить треки первого альбома массива
```js
let albums = Source.getArtistsAlbums(artists, {
    groups: 'album',
});
let albumTracks = Source.getAlbumTracks(albums[0]);
```

Пример 2 - Получить треки из всех альбомов
```js
let albums = Source.getArtistsAlbums(artists, {
    groups: 'album',
});
let tracks = [];
albums.forEach((album) => Combiner.push(tracks, Source.getAlbumTracks(album)));
```

### getArtists

Возвращает массив исполнителей согласно заданным `paramsArtist`.

Аргументы
- (объект) `paramsArtist` - перечень критериев отбора исполнителей. Объект соответствует описанию из [getArtistsTracks](/func?id=getartiststracks) в части исполнителя.

Пример 1 - Получить массив отслеживаемых исполнителей
```js
let artists = Source.getArtists({
    followed_include: true,
});
```

### getArtistsAlbums

Возвращает массив со всеми альбомами указанных исполнителей.

Аргументы
- (массив) `artists` - массив исполнителей
- (объект) `paramsAlbum` - перечень критериев отбора альбомов. Объект соответствует описанию из [getArtistsTracks](/func?id=getartiststracks) в части альбома.

Пример 1 - Получить массив синглов одного исполнителя
```js
let artist = Source.getArtists({
    followed_include: false,
    include: [ 
        { id: 'abc', name: 'Avril' }, 
    ],
});
let albums = Source.getArtistsAlbums(artist, {
    groups: 'single',
});
```

### getArtistsTopTracks

Возвращает топ треков исполнителя в виде массива. До 10 треков на исполнителя.

Аргументы
- (массив) `artists` - массив исполнителей. Значимо только `id`.
- (бул) `isFlat` - если `false` результат содержит треки в отдельном массиве для каждого исполнителя. Если `true` все треки в одном массиве. По умолчанию `true`.

Пример 1 - `isFlat = true`
```js
let tracks = Source.getArtistsTopTracks(artists);
tracks[0]; // первый трек первого исполнителя
tracks[10]; // первый трек второго исполнителя, если у первого 10 треков
```

Пример 2 - `isFlat = false`
```js
let tracks = Source.getArtistsTopTracks(artists, false);
tracks[0][0]; // первый трек первого исполнителя
tracks[1][0]; // первый трек второго исполнителя
```

### getArtistsTracks

Возвращает массив треков исполнителей согласно заданным `params`.

!> В выборку попадает множество альбомов. Особенно при большом количестве отслеживаемых исполнителей (100+). Для сокращения времени выполнения используйте фильтры для исполнителя и альбома. Можно указать случайный выбор N-количества.

?> У Spotify API есть баг. Не смотря на явное указание страны из токена, некоторые альбомы могут дублироваться (для разных стран). Используйте удаление дубликатов для треков.

Аргументы
- (объект) `params` - перечень критериев отбора исполнителей и их треков

| Ключ | Тип | Описание |
|-|-|-|
| followed_include | бул | При `true` включает отслеживаемых исполнителей. При `false` исполнители берутся только из `include` |
| include | массив | Выборка исполнителей по `id` для получения альбомов. Ключ `name` для удобства и необязателен.  |
| exclude | массив | Выборка исполнителей по `id` для исключения исполнителей из выборки. Использовать в комбинации с `followed_include` |
| popularity | объект | Диапазон популярности исполнителя |
| followers | объект | Диапазон количества фолловеров исполнителя |
| genres | массив | Перечень жанров. Если хотя бы один есть, исполнитель проходит фильтр.  |
| ban_genres | массив | Перечень жанров для блокировки. Если хотя бы один есть, исполнитель удаляется из выборки. |
| groups | строка | Тип альбома. Допустимо: `album`, `single`, `appears_on`, `compilation` |
| release_date | объект | Дата выхода альбома. Относительный период при `sinceDays` и `beforeDays`. Абсолютный период при `startDate` и `endDate` |
| _limit | число | Если указано, выбирается случайное количество указанных элементов (artist, album, track) |

Пример объекта `params` со всеми ключами
```js
{
    artist: {
        followed_include: true,
        popularity: { min: 0, max: 100 },
        followers: { min: 0, max: 100000 },
        artist_limit: 10,
        genres: ['indie'],
        ban_genres: ['rap', 'pop'],
        include: [
            { id: '', name: '' }, 
            { id: '', name: '' },
        ],
        exclude:  [
            { id: '', name: '' }, 
            { id: '', name: '' },
        ],
    },
    album: {
        groups: 'album,single',
        release_date: { sinceDays: 6, beforeDays: 0 },
        // release_date: { startDate: new Date('2020.11.30'), endDate: new Date('2020.12.30') },
        album_limit: 10,
        track_limit: 1,
    }
}
```

Пример 1 - Получить треки из синглов отслеживаемых исполнителей, вышедших за последнюю неделю включая сегодня. Исключить несколько исполнителей.
```js
let tracks = Source.getArtistsTracks({
    artist: {
        followed_include: true,
        exclude:  [
            { id: 'abc1', name: '' }, 
            { id: 'abc2', name: '' },
        ],
    },
    album: {
        groups: 'single',
        release_date: { sinceDays: 7, beforeDays: 0 },
    },
});
```

Пример 2 - Получить треки из альбомов и синглов за неделю десяти отслеживаемых исполнителей, выбранных случайным образом. Исполнители с не более чем 10 тысячами подписчиков. Только один трек из альбома.
```js
let tracks = Source.getArtistsTracks({
    artist: {
        followed_include: true,
        artist_limit: 10,
        followers: { min: 0, max: 10000 },
    },
    album: {
        groups: 'album,single',
        track_limit: 1,
        release_date: { sinceDays: 7, beforeDays: 0 },
    },
});
```

Пример 3 - Получить треки из альбомов и синглов указанных исполнителей
```js
let tracks = Source.getArtistsTracks({
    artist: {
        followed_include: false,
        include:  [
            { id: 'abc1', name: '' }, 
            { id: 'abc2', name: '' },
        ],
    },
    album: {
        groups: 'album,single',
    },
});
```

### getCategoryTracks

Возвращает массив треков из плейлистов указанной категории. Сортировка плейлистов по популярности. [Список категорий](/guide?id=Категории-плейлистов).

Аргументы
- (строка) `category_id` - имя категории.
- (объект) `params` - дополнительные параметры.

Описание `params`
- (число) `limit` - ограничить число выбираемых плейлистов. Максимум 50, по умолчанию 20.
- (число) `offset` - пропустить указанное число треков. По умолчанию 0.
- (строка) `country` - название страны, в которой смотреть плейлисты категории. Например, `RU` или `AU`.

Пример 1 - Получить треки второй десятки плейлистов категории "фокус" из Австралии.
```js
let tracks = Source.getCategoryTracks('focus', { limit: 10, offset: 10, country: 'AU' });
```

Пример 2 - Получить треки 20 плейлистов в категории вечеринки.
```js
let tracks = Source.getCategoryTracks('party');
```

### getFollowedTracks

Возвращает массив треков отслеживаемых плейлистов и/или личных плейлистов указанного пользователя.

?> Если нужно выполнить разные действия над источником, создайте копию массива [sliceCopy](/func?id=slicecopy) вместо новых запросов к Spotify через getFollowedTracks.

Аргументы
- (объект) `params` - аргументы отбора плейлистов.

Описание ключей
- (строка) `type` - тип выбираемых плейлистов. По умолчанию `followed`.
- (строка) `userId` - [идентификатор пользователя](#идентификатор). Если не указан, устанавливается `userId` авторизированного пользователя, то есть ваш.
- (число) `limit` - если используется, плейлисты выбираются случайным образом.
- (массив) `exclude` - перечень плейлистов, которые нужно исключить. Значимо только `id`. Значение `name` необязательно, нужно лишь для понимания какой это плейлист. Можно обойтись комментарием.

|type|Выбор|
|-|-|
| owned | Только личные плейлисты |
| followed | Только отслеживаемые плейлисты |
| all | Все плейлисты |

Полный объект `params`
```js
{
    type: 'followed',
    userId: 'abc',
    limit: 2,
    exclude: [
        { name: 'playlist 1', id: 'abc1' },
        { id: 'abc2' }, // playlist 2
    ],
}
```

Пример 1 - Получить треки только из моих отслеживаемых плейлистов.
```js
// Все значения по умолчанию, аргументы не указываются
let tracks = Source.getFollowedTracks();

// Тоже самое с явным указанием типа плейлистов
let tracks = Source.getFollowedTracks({
    type: 'followed',
});
```

Пример 2 - Получить треки только двух случайно выбранных личных плейлистов пользователя `example`, исключая несколько плейлистов по их id. 
```js
let tracks = Source.getFollowedTracks({
    type: 'owned',
    userId: 'example',
    limit: 2, 
    exclude: [
        { id: 'abc1' }, // playlist 1
        { id: 'abc2' }, // playlist 2
    ],
});
```

!> Следует избегать пользователей со слишком большим количеством плейлистов. Например, `glennpmcdonald` почти с 5 тысячами плейлистов. Ограничение связано с квотой на выполнение в Apps Script. За отведенное время не удастся получить такой объем треков. Подробнее в [описании ограничений](/overview?id=Ограничения).


### getListCategory

Возвращает массив допустимых категорий для [getCategoryTracks](/func?id=getcategorytracks).

Аргументы
- (объект) `params` - параметры отбора категорий.

Описание `params`
- (число) `limit` - ограничить число выбираемых категорий. Максимум 50, по умолчанию 20.
- (число) `offset` - пропустить указанное число категорий. По умолчанию 0. Используется для получения категорий после 50+.
- (строка) `country` - название страны, в которой смотреть категории. Например, `RU` или `AU`. Если нет, глобально доступные. Но возможна ошибка доступности. Чтобы не получить ошибки, указывайте одинаковые `country` для списка категорий и запроса плейлистов. 

Пример 1 - Получить треки 10 плейлистов из случайной категории
```js
let listCategory = Source.getListCategory({ limit: 50, country: 'RU' });
let category = Selector.sliceRandom(listCategory, 1);
let tracks = Source.getCategoryTracks(category[0].id, { limit: 10, country: 'RU' });
```

### getPlaylistTracks

Возвращает массив треков из одного плейлиста. Аналогично [getTracks](/func?id=gettracks) с одним плейлистом.

Аргументы
- (строка) `name` - имя плейлиста.
- (строка) `id` - [идентификационный номер плейлиста](/guide?id=Плейлист).
- (строка) `user` - [идентификационный номер пользователя](/guide?id=Пользователь). По умолчанию ваш.
- (число) `count` - количество выбираемых треков.
- (булево) `inRow` - режим выбора. Если нет ключа или `true`, выбор первых `count` элементов, иначе случайный выбор

Пример 1 - Получить треки одного плейлиста
```js
let tracks = Source.getPlaylistTracks('Заблокированный треки', 'abcdef');
```

Пример 2 - Случайный выбор 10 треков
```js
// имя плейлиста и пользователя можно оставить пустым, если известно id плейлиста
let tracks = Source.getPlaylistTracks('', 'id', '', 10, false);
```

### getRecomTracks

Возвращает массив рекомендованных треков по заданным параметрам (до 100 треков). Для новых или малоизвестных исполнителей/треков возможно будет недостаточно накопленных данных для генерации рекомендаций. 

Аргументы
- (объект) `queryObj` - параметры для отбора рекомендаций.

Допустимые параметры
- limit - количество треков. Максимум 100.
- seed_* - до **5 значений** в любых комбинациях:
  - seed_artists - [идентификаторы исполнителей](/guide?id=Идентификатор), разделенных запятой.
  - seed_tracks - [идентификаторы треков](/guide?id=Идентификатор), разделенных запятой.
  - seed_genres - жанры, разделенные запятой. Допустимые значения смотреть [здесь](/guide?id=Жанры-для-отбора-рекомендаций).
- max_* - предельное значение одной из [особенностей (features) трека](/guide?id=Особенности-трека-features).
- min_* - минимальное значение одной из [особенностей (features) трека](/guide?id=Особенности-трека-features).
- target_* - целевое значение одной из [особенностей (features) трека](/guide?id=Особенности-трека-features). Выбираются наиболее близкие по значению.

?> Кроме того, в `features` доступен ключ `populatiry`. Например, `target_popularity`. В документации к API Spotify это скрыто.

!> При указании конкретного жанра в `seed_genres` необязательно придут треки данного жанра. Такой сид является отправной точкой для рекомендаций.

Пример объекта с параметрами
```js
let queryObj = {
    seed_artists: '',
    seed_genres: '',
    seed_tracks: '',
    max_*: 0,
    min_*: 0,
    target_*: 0,
};
```

Пример 1 - Получить рекомендации по жанру инди и альтернативы с позитивным настроением:
```js
let tracks = Source.getRecomTracks({
    seed_genres: 'indie,alternative',
    min_valence: 0.65,
});
```

Пример 2 - Получить рекомендации в жанре рок и электроники на основе 3 случайных любимых исполнителей (до 5 значений).
```js
let savedTracks = Source.getSavedTracks();
Selector.keepRandom(savedTracks, 3);

let artistIds = savedTracks.map(track => track.artists[0].id);

let tracks = Source.getRecomTracks({
    seed_artists: artistIds.join(','),
    seed_genres: 'rock,electronic'
});
```

Пример 3 - Подстановка жанров исполнителей. Во всех запросах `craftTracks` будут один раз случайно отобранные жанры. При следующем запуске уже другие.
```js
let artists = Source.getArtists({ followed_include: false });
let genres = Array.from(new Set(artists.reduce((genres, artist) => {
  return Combiner.push(genres, artist.genres);
}, [])));
let recomTracks = Source.craftTracks(artists, {
  key: 'seed_artists',
  query: {
    seed_genres: Selector.sliceRandom(genres, 3).join(','),
    min_popularity: 20,
  }
});
```

### getRelatedArtists

Возвращает массив похожих исполнителей по данным Spotify.

Аргументы
- (массив) `artists` - перечень исполнителей, для которых получить похожих. Значимо только `id`.
- (бул) `isFlat` - если `false` результат содержит исполнителей в отдельном массиве. Если `true` все исполнители в одном массиве. По умолчанию `true`.

Пример 1 - `isFlat = true`
```js
let relatedArtists = Source.getRelatedArtists(artists);
relatedArtists[0]; // первый исполнитель
relatedArtists[10]; // 11 исполнитель
```

Пример 2 - `isFlat = false`
```js
let relatedArtists = Source.getRelatedArtists(artists, false);
relatedArtists[0][0]; // первый исполнитель, похожие на первого из источника
relatedArtists[1][0]; // первый исполнитель, похожие на второго из источника
```

### getSavedAlbumTracks

Возвращает массив треков со всех сохраненных альбомов. Можно задать выбор альбомов случайным образом.

Аргументы:
- (число) `limit` - если используется, альбомы выбираются случайно до указанного значения.

Пример 1 - Получить треки трех случайных альбомов
```js
let tracks = Source.getSavedAlbumTracks(3);
```

Пример 2 - Получить треки из всех сохраненных альбомов
```js
let tracks = Source.getSavedAlbumTracks();
```

### getSavedTracks

Возвращает массив любимых треков (лайков).

Аргументов нет.

?> Если у вас много любимых треков и в скрипте нужно выполнить разные действия над ними, создайте копию массива [sliceCopy](/func?id=slicecopy) вместо новых запросов к Spotify.

Пример 1 - Получить массив любимых треков.
```js
let tracks = Source.getSavedTracks();
```

### getTopArtists

Возвращает топ исполнителей за выбранный период. До 98 исполнителей. 

Аргументы
- (строка) `timeRange` - период. По умолчанию `medium`. Возможные значения приведены в [getTopTracks](/func?id=gettoptracks).

Пример 1 - Получить топ треков от топ 10 исполнителей
```js
let artists = Source.getTopArtists('long');
Selector.keepFirst(artists, 10);
let tracks = Source.getArtistsTopTracks(artists);
```

### getTopTracks

Возвращает массив треков с топом прослушиваний за выбранный период. До 98 треков.

Аргументы
- (строка) `timeRange` - период. По умолчанию `medium`.

|timeRange|Период|
|-|-|
| short | Примерно последний месяц |
| medium | Примерно последние 6 месяцев |
| long | За несколько лет |

!> Такие треки не содержат информации о дате добавления. При использовании [rangeDateRel](/func?id=rangedaterel) или [rangeDateAbs](/func?id=rangedateabs) им присваивается дата 01.01.2000

Пример 1 - Получить топ за последний месяц.
```js
let tracks = Source.getTopTracks('short');
```

Пример 2 - Получить топ за несколько лет.
```js
let tracks = Source.getTopTracks('long');
```

### getTracks

Возвращает массив треков из одного и более плейлистов.

Аргументы
- (массив) `playlistArray` - один и более плейлист. 

Формат *одного* плейлиста
- `id` - [идентификационный номер плейлиста](/guide?id=Плейлист).
- `userId` - [идентификационный номер пользователя](/guide?id=Пользователь).
- `name` - имя плейлиста.
- `count` - количество выбираемых треков.
- `inRow` - режим выбора. Если нет ключа или `true`, выбор первых `count` элементов, иначе случайный выбор

| id | name | userId | Действие |
|:-:|:-:|:-:|:-|
| ✓ | ☓ | ☓ | Взять плейлист с указанным id |
| ☓ | ✓ | ☓ | Поиск плейлиста по имени среди ваших |
| ☓ | ✓ | ✓ | Поиск плейлиста по имени у пользователя |

?> Рекомендуется всегда указывать `id` и `name`. Наиболее быстрый и удобный способ.

!> Если указано `name` без `id` и есть несколько плейлистов с таким именем, вернутся треки из первого встретившегося. Когда плейлист не найден, вернется пустой массив.

Пример 1 - Получить треки двух плейлистов по `id`. Значение `name` необязательно. Указывается для удобства.
```js
let tracks = Source.getTracks([
  { name: 'Главные хиты', id: '37i9dQZF1DX12G1GAEuIuj' },
  { name: 'Кардио', id: '37i9dQZF1DWSJHnPb1f0X3' },
]);
```

Пример 2 - Получить треки личных плейлистов The Best и Саундтреки.
```js
let tracks = Source.getTracks([
  { name: 'The Best' },
  { name: 'Саундтреки' },
]);
```

Пример 3 - Получить треки плейлиста с названием mint у пользователя spotify.
```js
let tracks = Source.getTracks([
  { name: 'mint', userId: 'spotify' },
]);
```

### getTracksRandom

Возвращает массив треков из одного и более плейлистов. Плейлисты выбираются случайным образом. 

Аргументы
- (массив) `playlistArray` - один и более плейлист. Аналогично [getTracks](/func?id=gettracks).
- (число) `countPlaylist` - количество случайно выбираемых плейлистов. По умолчанию один.

Пример 1 - Получить треки одного случайно выбранного плейлиста из трех.
```js
let tracks = Source.getTracksRandom([
  { name: 'Главные хиты', id: '37i9dQZF1DX12G1GAEuIuj' },
  { name: 'Кардио', id: '37i9dQZF1DWSJHnPb1f0X3' },
  { name: 'Темная сторона', id: '37i9dQZF1DX73pG7P0YcKJ' },
]);
```

Пример 2 - Получить треки двух случайно выбранных плейлистов из трех.
```js
let playlistArray = [
  { name: 'Главные хиты', id: '37i9dQZF1DX12G1GAEuIuj' },
  { name: 'Кардио', id: '37i9dQZF1DWSJHnPb1f0X3' },
  { name: 'Темная сторона', id: '37i9dQZF1DX73pG7P0YcKJ' },
];
let tracks = Source.getTracksRandom(playlistArray, 2);
```

### mineTracks

Возвращает массив треков, найденных при поиске плейлистов, альбомов или треков по ключевым словам. Из результата удаляются дубликаты.

Аргументы
- (объект) `params` - параметры поиска.

Описание `params`
- (строка) `type` - тип поиска. Допустимо: `playlist`, `album`, `track`. По умолчанию `playlist`. При `track` можно использовать [расширенный поиск](https://support.spotify.com/by-ru/article/search/).
- (массив) `keyword` - перечень ключевых слов для поиска элементов.
- (число) `requestCount` - количество запросов на одно ключевое слово. С одного запроса 50 элементов, если они есть. Максимум 40 запросов. По умолчанию один.
- (число) `itemCount` - количество выбираемых элементов из всех найденных на одно ключевое слово. По умолчанию три.
- (бул) `inRow` - если не указано или `false`, элементы выбираются случайно. Если `true` берутся первые `N` элементов (по значению `itemCount`).
- (число) `popularity` - минимальное значение популярности трека. По умолчанию ноль.
- (объект) `followers` - диапазон количества подписчиков плейлиста (границы включительно). Фильтр до выбора `itemCount`. Используйте только с малым количеством `requestCount` при `type = playlist`. 

!> Необходимо соблюдать баланс значений в `params`. Несколько больших значений могут занять много времени выполнения и совершить много запросов. Выясняйте на практике приемлемые комбинации.

> Можно вывести количество совершенных запросов. Добавьте строчку в конец функции: 
> `console.log('Число запросов', CustomUrlFetchApp.getCountRequest());`

Пример 1 - Выбор 5 случайных плейлистов по каждому ключевому слову с популярностью треков от 70. С ограниченным количеством подписчиков плейлистов.
```js
let tracks = Source.mineTracks({
    keyword: ['synth', 'synthpop', 'rock'],
    followers: { min: 2, max: 1000 },
    itemCount: 5,
    requestCount: 3,
    popularity: 70,
});
```

Пример 2 - Выбор 10 первых плейлистов по ключевому слову с любой популярностью треков
```js
let tracks = Source.mineTracks({
    keyword: ['indie'],
    itemCount: 10,
    inRow: true,
});
```

Пример 3 - Выбор треков из случайных альбомов
```js
let tracks = Source.mineTracks({
    type: 'album',
    keyword: ['winter', 'night'],
});
```

Пример 4 - Выбор треков в жанре инди за 2020 год
```js
let tracks = Source.mineTracks({
    type: 'track',
    keyword: ['genre:indie + year:2020'],
});
```

## Yandex

Модуль по работе с Яндекс.Музыкой

### getAlbums

Возвращает массив альбомов из подписок Яндекс.Музыки указанного пользователя. Примечание смотреть в [getTracks](/func?id=gettracks-1).

Аргументы
- (строка) `owner` - логин пользователя Яндекс.Музыки
- (число) `limit` - количество выбираемых альбомов. Если не указано, все.
- (число) `offset` - смещение от первого альбома. Например, `limit` = 50 и `offset` = 50 вернут альбомы от 50-го до 100-го.

Пример 1 - Получить все альбомы из подписок пользователя
```js
let albums = Yandex.getAlbums('owner');
```

### getArtists

Возвращает массив исполнителей из подписок Яндекс.Музыки указанного пользователя. Поиск аналога в базе Spotify по имени исполнителя. Внимание на [ограничения](/overview?id=Ограничения). Один исполнитель = один запрос поиска. Указанный пользователь должен иметь публично доступную библиотеку. Настройка находится [здесь](https://music.yandex.ru/settings/other).

!> Поиск осуществляется по наилучшему первому совпадению. Поэтому могут появляться "артефакты". Например, вместо исполнителя [Shura](https://open.spotify.com/artist/1qpR5mURxk3d8f6mww6uKT) найдется [Шура](https://open.spotify.com/artist/03JHGoUoM1LQmuXqknBi5P).

Аргументы
- (строка) `owner` - логин пользователя Яндекс.Музыки
- (число) `limit` - количество выбираемых исполнителей. Если не указано, все.
- (число) `offset` - смещение от первого исполнителя. Например, `limit` = 50 и `offset` = 50 вернут исполнителей от 50-го до 100-го.

Пример 1 - Подписаться на 50 последних исполнителей с Яндекса в Spotify. Можно запускать через триггер. Таким образом получить одностороннюю синхронизацию.
```js
let artists = Yandex.getArtists('owner', 50);
Library.followArtists(artists);
```

### getTracks

Возвращает массив треков плейлиста Яндекс.Музыки. Поиск аналога в базе Spotify по имени исполнителя и названию трека. Внимание на [ограничения](/overview?id=Ограничения). Один трек = один запрос поиска. Указанный пользователь должен иметь публично доступную библиотеку. Настройка находится [здесь](https://music.yandex.ru/settings/other). Кроме того, сам плейлист должен быть публичным (у них есть локальная приватность).

!> Поиск осуществляется по наилучшему первому совпадению. Поэтому могут появляться "артефакты". Например, треки, которые являются полными синонимами или попытка найти трек, которого нет в базе.

Обязательные аргументы
- (строка) `owner` - логин пользователя Яндекс.Музыки
- (строка) `kinds` - номер плейлиста
- (число) `limit` - количество выбираемых треков. Если не указано, все.
- (число) `offset` - смещение от первого трека. Например, `limit` = 50 и `offset` = 50 вернут треки от 50-го до 100-го.

Аргументы берутся из ссылки на плейлист. Например, для ссылки `https://music.yandex.ru/users/yamusic-daily/playlists/46484894`: логин это `yamusic-daily`, номер это `46484894`.

Пример 1 - Создать Плейлист дня из треков Яндекс.Музыки
```js
 Playlist.saveWithReplace({
     // id: 'ваше id', // после первого создания
     name: 'Плейлист дня',
     tracks: Yandex.getTracks('yamusic-daily', 'ваше id плейлиста дня'),
     randomCover: 'update',
 });
```
