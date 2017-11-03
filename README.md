# Live blob (Живая капля)

Позволяет добавить на статическое изображение "живую" каплю воды

![alt text](https://raw.githubusercontent.com/mrzigo/live-blob-water/master/logo.gif)
![alt text](https://raw.githubusercontent.com/mrzigo/live-blob-water/master/logo2.gif)

### 0. install
```
   npm install --save git+https://github.com/mrzigo/live-blob-water.git
```
### 1. Require
```
  import Blob from 'live-blob-water' // or var Blob = require('live-blob-water');
```

### 2. Using
Входящие данные:
```
const blobData = {
  cyclical: true,            // капля претекает из формы_1 в форму_2 (об этом ниже (1))
  el: $('.my_picture'),      // jquery элемент в который будет добавлен (append) canvas с искаженным изображением
  speed: 100,                // скорость перетекания (setInterval), default = 100
  src: "/static/s-drop.png", // изображение которое требуется искажать(предварительно вырезать из общей картинки)
  width: 100,                // ширина canvas, default = 100
  height: 100,               // высота canvas, default = 100
  left: "175px",             // отступ, default = 0
  top: "168px",              // отступ, default = 0
  center: [50, 50],          // центр капли, default = [width/2, height/2]
  lightVector: [60, 60],     // конечная точка вектора света (из центра капли в эту точку), длинна вектора влияет на длинну тени
  glare: true,               // рисовать блик, default = true
  sectors: [                 // форма капли представляет собой массив кривых Безье, заданых тремя точками
    {
      a: [[50, 20], [50, 20], 0, 0.1],  // начальная точка кривой (об этом ниже (2))
      b: [[85, 10], [81, 24], 0, 0.05], // опорная точка кривой (изгиб)
      c: [[75, 40], [80, 50], 0, 0.05], // конечная точка кривой
    },
    {
      a: null,                          // эта точка используется из предыдущего сектора
      b: [[75, 80], [81, 74], 0, 0.1],
      c: [[45, 85], [50, 80], 0, 0.1],
    },
    {
      a: null,                         // эта точка используется из предыдущего сектора
      b: [[30, 90], [24, 80], 0, 0.05],
      c: [[30, 60], [20, 50], 0, 0.05],
    },
    {
      a: null,                         // эта точка используется из предыдущего сектора
      b: [[15, 10], [24, 24], 0, 0.1],
      c: null,                         // эта точка используется из самого первого сектора (используется точка sectors[0].a)
    },
  ]
}
```
(1), (2) - форма капли задается через сектора, сектора задаются следующим образом:
```
sectors[0].a = [A, A1, status, step]
```
где **A** - начальная точка, **A1** - конечная точка, **status** - t ∈ [0,1], переход от точки A до точки A1 происходит по прямой проходящей через эти точки, t - отражает положение текущей точки, шаг перехода задается через **step**
непосредственно использование:
```
const myBlob = new Blob(blobData)
```
сразу же произойдет автоматическая загрузка картинки по URL src и вставка капли в el, а так же запуска анимации, который прекратиться сразу как вся форма перейдет в конечные точки и если cyclical === false, иначе начнется обратная трансформация в начальную форму

### 3. Change
```
  git clone https://github.com/mrzigo/live-blob-water
  cd live-blob-water
  npm i
  // выполнить изменения которые хотите внести в проект
  npm run build
  // использовать файл **dist/live-blob-water.js**
```

### 4. Схема преобразования
![alt text](https://raw.githubusercontent.com/mrzigo/live-blob-water/master/schema.png)

Красная форма перетекает в зеленую, первый сектор можно задать так:
sectors[0] = {
  a: [A, A', 0, 0.01],
  b: [E, E', 0, 0.01],
  c: [B, B', 0, 0.01],
}
на рисунке выше должно быть понятно как происходит анимация и как задовать остальные сектора. секторов не обязательно 4, может быть больше, может меньше.

линза накладывается по прямой от центра (т.О) до точки лежащей на кривой Безье, на схеме отрезок: ОК, коэффициент преобразования следующий:
```
(r / length) * (r / length) * 0.2 + 1
```
где r - параметр на отрезке [0,1], length - длинна отрезка OK
подробности в коде метода lensEffect (src/index.js)
