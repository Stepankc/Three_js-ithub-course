## (AmbientLight)[https://threejs.org/docs/index.html#api/en/lights/AmbientLight]

`AmbientLight` применяет всенаправленное освещение ко всем геометриям сцены. Первый параметр — это цвет, а второй параметр — интенсивность. Что касается материалов, вы можете установить свойства непосредственно при создании экземпляра или изменить их после:

```javascript
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

// Equals
const ambientLight = new THREE.AmbientLight();
ambientLight.color = new THREE.Color(0xffffff);
ambientLight.intensity = 0.5;
scene.add(ambientLight);
```

Как и в случае с материалами, вы можете добавить свойства в `lil.gui`.

```javascript
gui.add(ambientLight, "intensity").min(0).max(1).step(0.001);
```

Если у вас есть только `AmbientLight`, вы получите тот же эффект, что и для `MeshBasicMaterial`, потому что все грани геометрических фигур будут освещены одинаково.

В реальной жизни, когда вы освещаете объект, стороны объектов, противоположные свету, не будут полностью черными, потому что свет отражается от стен и других объектов. Отражение света не поддерживается в Three.js из-за производительности, но вы можете использовать окружающий свет, чтобы имитировать отражение света.

## (DirectionalLight)[https://threejs.org/docs/index.html#api/en/lights/DirectionalLight]

`DirectionalLight` будет иметь эффект, подобный солнечному.

Первый параметр `color`, а второй `intensity`:

```javascript
const directionalLight = new THREE.DirectionalLight(0x00fffc, 0.4);
scene.add(directionalLight);
```

По умолчанию кажется, что свет будет исходить сверху. Чтобы изменить это, вы должны переместить весь источник света, используя свойство `position`, как если бы это был обычный объект.

```javascript
directionalLight.position.set(1, 0.25, 0);
```

## (HemisphereLight)[https://threejs.org/docs/index.html#api/en/lights/HemisphereLight]

`HemisphereLight` похож на `AmbientLight`, но отличается цветом неба от цвета земли.

Первый параметр `color` соответствует цвету неба, второй параметр `groundColor`, а третий параметр `intensity`:

```javascript
const hemisphereLight = new THREE.HemisphereLight(0xff0000, 0x0000ff, 0.3);
scene.add(hemisphereLight);
```

## (PointLight)[https://threejs.org/docs/index.html#api/en/lights/PointLight]

`PointLight` - слабый источник света. Источник света бесконечно мал, и свет распространяется равномерно во всех направлениях.

Первый параметр это `color`, а второй `параметр` `intensity`:

```javascript
const pointLight = new THREE.PointLight(0xff9000, 0.5);
scene.add(pointLight);
```

вы можете управлять расстоянием затухания и скоростью его затухания с помощью свойств `distance` и `decay`. Вы можете установить их в параметрах класса в качестве третьего и четвертого параметров или в свойствах экземпляра:

```javascript
const pointLight = new THREE.PointLight(0xff9000, 0.5, 10, 2);
```

## (RectAreaLight)[https://threejs.org/docs/index.html#api/en/lights/RectAreaLight]

`RectAreaLight` работает так же, как большие прямоугольные огни, которые вы можете видеть на съемочной площадке. Это смесь направленного и рассеянного света. Первый параметр - это `color`, второй параметр - это `intensity`, третий параметр `width` прямоугольника, а четвертый параметр - его `height`:

```javascript
const rectAreaLight = new THREE.RectAreaLight(0x4e00ff, 2, 1, 1);
scene.add(rectAreaLight);
```

## (SpotLight)[https://threejs.org/docs/index.html#api/en/lights/SpotLight]

SpotLight - работает как фонарик Вот список его параметров:

- `color`: цвет
- `intensity`: сила
- `distance`: расстояние, на котором интенсивность падает до 0
- `angle`: насколько велик луч
- `penumbra`: насколько рассеянным является контур луча
- `decay`: как быстро тускнеет свет

```javascript
const spotLight = new THREE.SpotLight(
  0x78ff00,
  0.5,
  10,
  Math.PI * 0.1,
  0.25,
  1
);
spotLight.position.set(0, 2, 3);
scene.add(spotLight);
```

Вращать наш `SpotLight` немного сложнее. Экземпляр имеет свойство с именем `target`, которое является `Object3D` . Прожектор всегда смотрит на этот `target`. Но если вы попытаетесь изменить его положение, прожектор не сдвинется с места:

```javascript
spotLight.target.position.x = -0.75;
```

Это из-за того, что `target` не было на сцене. Просто добавьте `target` в сцену, и это должно сработать:

```javascript
spotLight.target.position.x = -0.75;
```

## Помощники

Позиционирование и ориентация огней затруднены. Чтобы помочь нам, мы можем использовать помощников. Поддерживаются только следующие помощники:

- (HemisphereLightHelper)[https://threejs.org/docs/index.html#api/en/helpers/HemisphereLightHelper]
- (DirectionalLightHelper)[https://threejs.org/docs/index.html#api/en/helpers/DirectionalLightHelper]
- (PointLightHelper)[https://threejs.org/docs/index.html#api/en/helpers/PointLightHelper]
- (RectAreaLightHelper)[https://threejs.org/docs/index.html#examples/en/helpers/RectAreaLightHelper]
- (SpotLightHelper)[https://threejs.org/docs/index.html#api/en/helpers/SpotLightHelper]

Чтобы использовать их, просто создайте экземпляр этих классов. Используйте соответствующие источники света в качестве параметра и добавьте их в сцену. Второй параметр позволяет изменять параметры помощника `size`:

```javascript
const hemisphereLightHelper = new THREE.HemisphereLightHelper(
  hemisphereLight,
  0.2
);
scene.add(hemisphereLightHelper);

const directionalLightHelper = new THREE.DirectionalLightHelper(
  directionalLight,
  0.2
);
scene.add(directionalLightHelper);

const pointLightHelper = new THREE.PointLightHelper(pointLight, 0.2);
scene.add(pointLightHelper);

const spotLightHelper = new THREE.SpotLightHelper(spotLight);
scene.add(spotLightHelper);
```

# P.S Свет лучше не использовать, тк он может серьезно влиять на производительность. Лучше делать `Bake` моделей в других программах (Blender).
# Идея в том, что вы впекаете свет в текстуру. К сожалению, вы не сможете перемещать источники света, потому что их нет, и вам, вероятно, понадобится много текстур.