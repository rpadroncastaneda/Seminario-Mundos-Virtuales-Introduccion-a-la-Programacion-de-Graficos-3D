# Seminario: Mundos Virtuales – Introducción a la Programación de Gráficos 3D

## 1. Funciones de traslación, rotación y escalado en Unity

En Unity, las transformaciones geométricas se realizan mediante el componente **Transform**.
Principales funciones:

```csharp
// Traslación
transform.Translate(x, y, z);

// Rotación
transform.Rotate(x, y, z);
transform.rotation = Quaternion.Euler(x, y, z);

// Escalado
transform.localScale = new Vector3(x, y, z);
```

Estas funciones modifican la **matriz de modelo** del objeto, que define su posición, orientación y escala en el espacio mundial.

---

## 2. Orden de transformaciones: traslación y rotación de la cámara

```csharp
// A) Trasladar y luego rotar
transform.Translate(2, 2, 2);
transform.Rotate(0, 30, 0);

// B) Rotar y luego trasladar
transform.Rotate(0, 30, 0);
transform.Translate(2, 2, 2);
```

El resultado **no es el mismo**.
Las transformaciones **no conmutan**:

- En A, la cámara se mueve en el sistema mundial y luego gira sobre su nuevo eje.
- En B, primero se rota (cambia su sistema local) y luego se traslada según sus ejes locales rotados.

> En coordenadas homogéneas, esto equivale a multiplicar matrices en distinto orden:
> `M_total = T * R ≠ R * T`.

---

## 3. Esfera parcialmente recortada por el volumen de vista

Para que una esfera de radio 1 quede parcialmente dentro del volumen de vista:

```csharp
Camera.main.nearClipPlane = 1f;
Camera.main.farClipPlane = 3f;
```

Si la esfera está centrada en `z = 2`, los planos del frustum recortan parte de ella.

---

## 4. Esfera fuera del campo de visión

Situar la esfera fuera del volumen de vista ajustando los planos de recorte:

```csharp
Camera.main.nearClipPlane = 1f;
Camera.main.farClipPlane = 1.5f;
```

La esfera a `z = 2` quedará completamente fuera de la vista y no será renderizada.

---

## 5. Modificar el ángulo de visión de la cámara

```csharp
Camera.main.fieldOfView = 90f; // Aumentar FOV
Camera.main.fieldOfView = 30f; // Disminuir FOV
```

- Aumentar el **FOV** amplía el campo de visión, los objetos parecen más pequeños y distantes.
- Disminuirlo reduce el campo de visión, los objetos se ven más grandes y cercanos.

---

## 6. Proyección ortográfica

La afirmación es **correcta**.
Para realizar la proyección al espacio 2D sin perspectiva se cambia:

```csharp
Camera.main.orthographic = true;
```

En este modo, no hay efecto de profundidad y los objetos mantienen su tamaño sin importar la distancia.

---

## 7. Rotaciones con quaternions

Las rotaciones se pueden expresar con cuaterniones:

```csharp
transform.rotation = Quaternion.Euler(0, 30, 0);
```

o concatenar rotaciones:

```csharp
transform.rotation *= Quaternion.Euler(0, 30, 0);
```

Los **quaternions** representan rotaciones en el espacio 3D sin los problemas de bloqueo de ejes (gimbal lock).

---

## 8. Matriz de proyección en perspectiva

Se puede obtener en tiempo de ejecución con:

```csharp
Matrix4x4 projMatrix = Camera.main.projectionMatrix;
Debug.Log(projMatrix);
```

Esta matriz transforma coordenadas del **espacio de vista** al **espacio de clip** y depende de:

- `fieldOfView`
- `aspect`
- `nearClipPlane`
- `farClipPlane`

---

## 9. Matriz de proyección ortográfica

Al cambiar a proyección ortográfica:

```csharp
Camera.main.orthographic = true;
Matrix4x4 orthoMatrix = Camera.main.projectionMatrix;
Debug.Log(orthoMatrix);
```

---

## 10. Matriz de transformación local ↔ mundial

```csharp
Matrix4x4 localToWorld = transform.localToWorldMatrix;
Matrix4x4 worldToLocal = transform.worldToLocalMatrix;
```

Estas matrices conectan el **sistema local del objeto** con el **sistema mundial**.

---

## 11. Matriz de cambio al sistema de referencia de vista

La **matriz de vista** transforma del sistema mundial al sistema de coordenadas de la cámara:

```csharp
Matrix4x4 viewMatrix = Camera.main.worldToCameraMatrix;
```

---

## 12. Matriz de proyección usada en ejecución

```csharp
Debug.Log(Camera.main.projectionMatrix);
```

Permite inspeccionar la proyección activa en un instante del render.

---

## 13. Matrices de modelo y vista de la escena

- **Modelo:** `transform.localToWorldMatrix`
- **Vista:** `Camera.main.worldToCameraMatrix`

Estas matrices se combinan para transformar los vértices de cada objeto al sistema de la cámara.

---

## 14. Rotación en Start y matriz mundial

```csharp
void Start() {
    transform.Rotate(0, 45, 0);
    Debug.Log(transform.localToWorldMatrix);
}
```

Muestra la matriz resultante del objeto respecto al sistema mundial tras aplicar la rotación.

---

## 15. Coordenadas con `Position(3,1,1)` y `Rotation(45,0,45)`

El transform combina:

- Traslación T(3, 1, 1)
- Rotación Rz(45°) · Rx(45°)

La matriz total:

```
M = T(3,1,1) · Rz(45°) · Rx(45°)
```

Define la posición y orientación del objeto respecto al mundo.

---

## 16. Escena base y script de depuración

**Elementos de la escena:**

- Cámara principal
- Plano base (suelo)
- Tres cubos (rojo, verde, azul) en distintas posiciones

**Script de depuración:**

```csharp
using UnityEngine;

public class MatrixDebugger : MonoBehaviour {
    public Transform redCube, greenCube, blueCube;

    void Update() {
        Matrix4x4 view = Camera.main.worldToCameraMatrix;
        Matrix4x4 proj = Camera.main.projectionMatrix;

        Debug.Log("==== MATRICES ====");
        Debug.Log("View:\n" + view);
        Debug.Log("Projection:\n" + proj);

        foreach (Transform cube in new Transform[]{redCube, greenCube, blueCube}) {
            Matrix4x4 model = cube.localToWorldMatrix;
            Vector3 vertex = new Vector3(0.5f, 0.5f, 0.5f);
            Vector4 transformed = proj * view * model * new Vector4(vertex.x, vertex.y, vertex.z, 1);
            Debug.Log(cube.name + " vertex transformed: " + transformed);
        }
    }
}
```

---

## 17. Recorrido de coordenadas de un vértice

**Ejemplo:** vértice `(0.5, 0.5, 0.5)` del cubo rojo

| Espacio       | Descripción                    | Ejemplo de coordenadas |
| ------------- | ------------------------------ | ---------------------- |
| Local         | Coordenadas propias del cubo   | (0.5, 0.5, 0.5, 1)     |
| World         | Multiplicación por `Model`     | `(M * v_local)`        |
| View / Camera | Multiplicación por `View`      | `(V * M * v)`          |
| Clip          | Aplicación de `Projection`     | `(P * V * M * v)`      |
| NDC           | Normalización dividiendo por w | `(x/w, y/w, z/w)`      |
| Viewport      | Mapeo a píxeles de pantalla    | `(x', y')`             |

Cada transformación corresponde a una etapa del **pipeline gráfico**.

---

## 18. Cambios en las matrices al mover o rotar

- **Mover o rotar un cubo:** cambia su matriz de **modelo (Model Matrix)**.
- **Rotar la cámara:** modifica la **matriz de vista (View Matrix)**.
- **Cambiar proyección ortográfica ↔ perspectiva:** altera la **matriz de proyección (Projection Matrix)**.

Comparando los valores en consola se puede observar cómo varían los coeficientes de cada matriz y su efecto en las coordenadas finales.

---

## Referencias

- Apuntes de clase: ResumenGraficos3D Geometría y ResumenGraficos3D Iluminacion
- Documentación oficial Unity:
  [https://docs.unity3d.com/ScriptReference/Transform.html](https://docs.unity3d.com/ScriptReference/Transform.html)
  [https://docs.unity3d.com/ScriptReference/Camera.html](https://docs.unity3d.com/ScriptReference/Camera.html)

---

**Autor:**
Adrián García Rodríguez, Kyliam Gabriel Chinea Salcedo, Roberto Padrón Castañeda y Cristóbal Jesús Sarmiento Rodríguez

**Asignatura:** Grado en Ingeniería Informática
Interfaces Inteligentes

**Fecha:** 05/11/2025
