Мы создали сцену, которую отрисовали. Это уже хороший прогресс, но в сейчас вам интересно анимировать свои творения и попробовать что-то новое.

## Автоматическое изменение разрешения окна

Чтобы изменить размер окна, нам сначала нужно знать, когда оно изменяется. Для этого вы можете прослушать `resize` событие в окне.

Добавьте `resize` сразу после `sizes` переменной:

```javascript
window.addEventListener("resize", () => {
  console.log("window has been resized");
});
```

Теперь когда мы откроем консоль, то увидим что при любом изменении окна, появляется сообщение в консоли.
Для начала удаляем `console.log` и добавляем обновление переменной `sizes`:

```javascript
window.addEventListener("resize", () => {
  // Update sizes
  sizes.width = window.innerWidth;
  sizes.height = window.innerHeight;
});
```

Теперь нужно изменить свойство камеры, о нём вы можете прочитать сами, а сейчас просто добавьте его:

```javascript
window.addEventListener("resize", () => {
  camera.updateProjectionMatrix();
});
```

Наконец, мы должны обновить `renderer`, и записать в него наши размеры окна:

```javascript
window.addEventListener("resize", () => {
  // Update renderer
  renderer.setSize(sizes.width, sizes.height);
});
```

## Pixel Ratio

У некоторых появиться своего рода размытый рендеринг и артефакты в виде лестниц по краям. Это происходит из-за того что, ваше соотношение пикселей больше 1

Чтобы получить соотношение пикселей экрана, которое вы можете использовать, введите в консоль `window.devicePixelRatio`
Для того чтобы обновить соотношение пикселей, нам нужно вызвать `renderer.setPixelRatio(...)`

Соотношение пикселей соответствует количеству физических пикселей на экране.
Теперь мы можем ограничить `PixelRatio`:

```javascript
// ...

```

Это делается для того чтобы, наша производительность не пострадала, так как на некоторых устройствах `pixelRatio` достигает 5, а больше двух мы даже не заметим.

Сейчас `pixelRatio` ставится при обновлении страницы, но что если мы перенесём вкладку на другой экран? Наш `pixelRatio` так и останется, значит нужно его изменять при любых действиях с вкладкой, для этого добавим изменение `pixelRatio` в нашу `resize` функцию:

```javascript
window.addEventListener("resize", () => {
  // Update renderer
  // ...
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
});
```

## Анимации

Анимация в `Three.js` , работают как стоп-движение. Вы перемещаете объекты и выполняете рендеринг. Затем вы перемещаете объекты еще немного и выполняете другой рендеринг. Чем больше вы перемещаете объекты между рендерами, тем быстрее они будут перемещаться. Скорость анимаций зависит от вашего монитора, большинство экранов имеют 60гц и если экран имеют большую частоту, то анимация будет быстрее. Из-за этого мы можем испытывать трудности со скоростью воспроизведения анимации на разных экранах

Сейчас мы воспользуемся методом из ванильного js `window.requestAnimationFrame(...)`

## Взаимодействие с [requestAnimationFrame](https://developer.mozilla.org/ru/docs/Web/API/window/requestAnimationFrame)

Основная цель `requestAnimationFrame` - не запускать код для каждого кадра.

`requestAnimationFrame` выполнит функцию, которую вы предоставляете, в следующем кадре.В конечном итоге наша функция будет выполняться в каждом кадре вечно и это то, что мы хотим.

Создайте функцию и вызовите эту функцию один раз. В этой функции используется `window.requestAnimationFrame(...)` для вызова этой же функции в следующем кадре:

```javascript
//Animate
const tick = () => {
  console.log("tick");

  window.requestAnimationFrame(tick);
};
//call func
tick();
```

С помощью `console.log("tick")` мы видим что функция вызывается для каждого кадра, Если вы проверите этот код на компьютерах с разной частотой, то заметите, что `tick` появляется с большей частотой

Теперь мы можете переместить `renderer.render(...)` внутрь этой функции и увеличить куб `rotation`:

```javascript
//Animate
const tick = () => {
  // Update objects
  mesh.rotation.y += 0.01;

  // Render
  renderer.render(scene, camera);

  // Call tick again on the next frame
  window.requestAnimationFrame(tick);
};

tick();
```

Теперь у нас появилась базовая анимация модели в `Three.js`.Но при запуске этого кода на компе с высокой частотой кадров, куб будет двигаться быстрее, а на низкой, медленее.

## Адаптация под разную частоту кадров ([Clock](https://threejs.org/docs/#api/en/core/Clock))

В `Three.js` есть `Clock`, которые будут обрабатывать вычисления времени.

Нам просто нужно создать экземпляр переменной `Clock` и использовать встроенные методы, такие как `getElapsedTime()`. Этот метод вернет столько секунд, сколько секунд прошло с момента создания часов.

Вы можете использовать это значение для поворота объекта:

```javascript
//Animate
const clock = new THREE.Clock();

const tick = () => {
  const elapsedTime = clock.getElapsedTime();

  // Update objects
  mesh.rotation.y = elapsedTime;

  // ...
};

tick();
```

Вы также можете использовать его для перемещения объектов с помощью свойства `position`. Если вы объедините это с `Math.cos(...)`, вы можете получить довольно хороший результат:

```javascript
//Animate
const clock = new THREE.Clock();

const tick = () => {
  const elapsedTime = clock.getElapsedTime();

  // Update objects
  mesh.position.y = Math.sin(elapsedTime);
};

tick();
```

Мы можем использовать этот метод для любого объекта 3D, например для камеры. Мы используем `lookAt(...)` который мы разбирали на прошлой паре и получаем интересный результат:

```javascript
//Animate
const clock = new THREE.Clock();

const tick = () => {
  const elapsedTime = clock.getElapsedTime();

  // Update objects
  camera.position.x = Math.cos(elapsedTime);
  camera.position.y = Math.sin(elapsedTime);
  camera.lookAt(mesh.position);

  // ...
};

tick();
```

## Анимация с помощью сторонних библиотек [GSAP](https://greensock.com/gsap/)

Так как gsap имеет встроенную `requestAnimationFrame` нам не нужно обновлять анимацию самим

## Группа объектов[Group](https://threejs.org/docs/#api/en/objects/Group)

группа обьектов(`Group`)- используется для создания домов, заднего фона и моделей которые состоят из нескольких объектов. Объекты в группе маштабируются одновременно и используют `scale`,`rotation`,`position` самой группы.
Давайте создадим Group и добавим его в сцену:

```javascript
const group = new THREE.Group();
group.scale.y = 3;
group.rotation.y = 0.6;
scene.add(group);
```

Теперь мы можем добавить объект в нашу группу, для этого создаем `mesh` и добавляем его в группу используя `add(...)`:

```javascript
const cube1 = new THREE.Mesh(
  new THREE.BoxGeometry(1, 1, 1),
  new THREE.MeshBasicMaterial({ color: 0xff0000 })
);
cube1.position.x = -1.5;
group.add(cube1);
```

закомментируйте `lookAt(...)` и вместо нашего ранее созданного куба, создайте 3 куба и добавьте их в группу. Затем примените преобразования к `group`

## Камеры

На прошлой паре мы создали `PerpectiveCamera`, но также есть и другие типы камер которые мы сейчас посмотрим

### [ArrayCamera](https://threejs.org/docs/?q=camera#api/en/cameras/ArrayCamera)

`ArrayCamera` используется для многократного рендеринга сцены с использованием нескольких камер. Каждая камера будет отображать определенную область сцены.[Пример](https://threejs.org/examples/#webgl_camera_array)

### [CubeCamera](https://threejs.org/docs/?q=camera#api/en/cameras/CubeCamera)

`CubeCamera` используется для получения рендеринга в каждом направлении (вперед, назад, влево, вправо, вверх и вниз) для создания рендеринга окружения.[Пример](https://threejs.org/examples/#webgl_materials_cubemap_dynamic)

### [OrthographicCamera](https://threejs.org/docs/?q=camera#api/en/cameras/OrthographicCamera)

`OrthographicCamera` используется для создания ортографических рендеров вашей сцены без перспективы.
[Пример](https://threejs.org/examples/#webgl_camera)

### [PerspectiveCamera](https://threejs.org/docs/?q=camera#api/en/cameras/PerspectiveCamera)

Перспективная камера - это та, которую мы уже использовали и смоделировали реальную камеру с перспективой.[Пример](https://threejs.org/examples/#webgl_loader_collada_skinning)

### [StereoCamera](https://threejs.org/docs/?q=camera#api/en/cameras/StereoCamera)

Стереокамера используется для визуализации сцены с помощью двух камер. Используется для создания 3D,VR и тд.[Пример](https://threejs.org/examples/#webgl_effects_anaglyph)

## [OrbitControls](https://threejs.org/docs/?q=orb#examples/en/controls/OrbitControls)

Для начала, нам нужно создать экземпляр переменной, используя класс `OrbitControls` . Хотя вы можете подумать, что можете использовать `THREE.OrbitControls`, но это не так :(
Класс `OrbitControls` является частью тех классов, которые по умолчанию недоступны в THREE. Для того чтобы получить к ней доступ вы должны указать путь сами:

```javascript
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
```

Теперь вы можете использовать `OrbitControls` без `THREE`

```javascript
const controls = new OrbitControls(camera, canvas);
```

Теперь вы можете перемещать камеру с помощью левой или правой мыши, а также прокручивать вверх или вниз для увеличения или уменьшения масштаба.

В `OrbitControls`, есть сглаживание `damping`.

Чтобы его включить, переключите свойство `controls` (`enableDamping`) на true.

Также для правильной работы элементы управления его необходимо обновлять при каждом кадре с помощью вызова `controls.update()`. Для этого используем нашу функцию `tick`:

```javascript
// Controls
const controls = new OrbitControls(camera, canvas);
controls.enableDamping = true;

// ...

const tick = () => {
  // ...

  // Update controls
  controls.update();

  // ...
};
```

Вы увидите, что управление стало намного более плавным.
Вы можете использовать множество других методов и свойств для настройки элементов управления, таких как скорость вращения, скорость масштабирования и тд.
