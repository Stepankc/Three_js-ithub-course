Теперь, когда у нас есть свет, нам нужны тени. Задняя часть объектов действительно находится в темноте, и это называется основной тенью. Чего нам не хватает, так это отбрасывания тени.

Тени всегда были проблемой для 3D-рендеринга в реальном времени.

Существует множество способов их реализации, и `Three.js` имеет встроенное решение.

Давайте попробуем понять основы рендера теней.

Когда вы делаете один рендеринг, `Three.js` сначала сделаем рендеринг для каждого источника света, который должен отбрасывать тени.
Во время этих световых рендеров [MeshDepthMaterial](https://threejs.org/docs/index.html#api/en/materials/MeshDepthMaterial) заменяет все материалы сетки.

Результаты сохраняются в виде текстур и именованных `shadow maps`.

Вы не увидите эти `shadow maps` напрямую, но они используются для каждого материала, который должен получать тени, и проецируются на геометрию.

## Как активировать тени

1. нам нужно активировать `shadow maps` на `renderer`:

```javascript
renderer.shadowMap.enabled = true;
```

2. нам нужно просмотреть каждый объект сцены и решить, может ли объект отбрасывать тень с помощью свойства `castShadow`, и может ли объект получать тень с помощью свойства `receiveShadow`.

```javascript
sphere.castShadow = true;

// ...
plane.receiveShadow = true;
```

Только эти источники света поддерживают тени:

- PointLight
- DirectionalLight
- SpotLight

```javascript
directionalLight.castShadow = true;
```

Теперь у нас должна появиться тень на плоскости. На данный момент это выглядит ужасно, но мы это исправим.

Если тень не появилась, то вы допустили ошибку. -\_-

## Оптимизация Shadow map

Как мы уже говорили `three.js` выполняет рендеринг, называемый `shadow maps`, для каждого источника света. Вы можете получить к ней доступ в console.log:

```javascript
console.log(directionalLight.shadow);
```

Что касается нашего рендеринга, нам нужно указать размер. По умолчанию размер карты теней указан только 512x512 из-за производительности. Мы можем улучшить его,но нам нужна степень 2, иначе ничего работать не будет:

```javascript
directionalLight.shadow.mapSize.width = 1024;
directionalLight.shadow.mapSize.height = 1024;
```

Three.js использует камеры для рендеринга `shadow maps`. Эти камеры обладают теми же свойствами, что и камеры, которые мы уже использовали. Это означает, что мы должны определить `near` и `far`.Это может исправить ошибки, из-за которых вы не видите тень или обрезанную тень.

Чтобы было легче исправлять всё, мы можем использовать `CameraHelper` о котором мы уже говорили на прошлых парах:

```javascript
const directionalLightCameraHelper = new THREE.CameraHelper(
  directionalLight.shadow.camera
);
scene.add(directionalLightCameraHelper);
```

Теперь мы видим как расположена наша камера,остаётся только подставить значения, которые соответствуют сцене:

```javascript
directionalLight.shadow.camera.near = 1;
directionalLight.shadow.camera.far = 6;
```

Также мы видим что у камеры слишком большая амплитуда, её можно изменить с помощью свойств `top`,`right`,`bottom`,`left`

```javascript
directionalLight.shadow.camera.top = 2;
directionalLight.shadow.camera.right = 2;
directionalLight.shadow.camera.bottom = -2;
directionalLight.shadow.camera.left = -2;
```

Чем меньше значения, тем точнее будет тень. Но если они будут слишком маленькими,то тени будут просто обрезаны.

Для того чтобы скрыть помощник:

```javascript
directionalLightCameraHelper.visible = false;
```

## Размытие тени

Размытием тени можно управлять с помощью свойства `radius`:

```javascript
directionalLight.shadow.radius = 10;
```

## Алгоритм отображения теней

К картам теней могут быть применены различные типы алгоритмов:

- `THREE.BasicShadowMap`: очень эффективная, но плохого качества.
- `THREE.PCFShadowMap`: менее производительная, но более плавные края.
- `THREE.PCFSoftShadowMap`: менее производительная, но еще более мягкие края.
- `THREE.VSMShadowMap`: менее производительная, больше ограничений, может привести к неожиданным результатам.

Чтобы изменить его, обновите свойство `renderer.shadowMap.type`. По умолчанию используется `THREE.PCFShadowMap`, но вы можете использовать `THREE.PCFSoftShadowMap` для лучшего качества.

```javascript
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
```

Имейте в виду, что свойство `radius` не работает с `THREE.PCFSoftShadowMap`, выбираем что-то одно.
