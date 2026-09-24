---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 2. Angular

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

## 2.1 Fundamentos

**Angular** es un framework SPA basado en TypeScript, con inyección de dependencias, enrutamiento, formularios reactivos y RxJS integrados. Desde Angular 14+ la tendencia es **standalone components** (sin NgModules) y desde Angular 16/17 **Signals** para reactividad fina.

### Bloques del framework

| Concepto | Para qué sirve |
|---|---|
| Component | Unidad de UI: clase + template + estilos |
| Directive | Modifica comportamiento del DOM (`*ngIf`, `*ngFor`, custom) |
| Pipe | Transforma datos en el template (`date`, `currency`, custom) |
| Service | Lógica reutilizable / acceso a datos, inyectable |
| Module / Standalone | Agrupa y declara dependencias |
| Router | Navegación, lazy loading, guards |

## 2.2 Componente típico (standalone + signals)

```typescript
import { Component, inject, signal, computed } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UsuarioService } from './usuario.service';

@Component({
  selector: 'app-usuarios',
  standalone: true,
  imports: [CommonModule],
  template: `
    <h2>Usuarios activos: {{ totalActivos() }}</h2>
    @if (cargando()) {
      <p>Cargando…</p>
    } @else {
      <ul>
        @for (u of usuarios(); track u.id) {
          <li>{{ u.nombre }}</li>
        }
      </ul>
    }
  `
})
export class UsuariosComponent {
  private readonly service = inject(UsuarioService);

  readonly usuarios = signal<Usuario[]>([]);
  readonly cargando = signal(true);
  readonly totalActivos = computed(() =>
    this.usuarios().filter(u => u.activo).length
  );

  ngOnInit(): void {
    this.service.listar().subscribe({
      next: data => { this.usuarios.set(data); this.cargando.set(false); },
      error: () => this.cargando.set(false)
    });
  }
}
```

## 2.3 Ciclo de vida 🎯

| Hook | Cuándo se ejecuta | Uso habitual |
|---|---|---|
| `ngOnChanges` | Cambia un `@Input` | Reaccionar a props |
| `ngOnInit` | Una vez, tras el primer bind | Carga inicial de datos |
| `ngDoCheck` | Cada ciclo de detección | Detección custom (usar con cuidado) |
| `ngAfterViewInit` | Vista hija lista | Acceso a `@ViewChild`, librerías DOM |
| `ngOnDestroy` | Al destruir | **Liberar suscripciones**, timers |

**Error clásico:** hacer la llamada HTTP en el constructor. El constructor es para inyectar; `ngOnInit` es para inicializar.

## 2.4 RxJS — lo mínimo que te van a preguntar 🎯

```typescript
// Buscador con debounce: el patrón que más piden explicar
this.form.get('q')!.valueChanges.pipe(
  debounceTime(300),              // espera a que deje de tipear
  distinctUntilChanged(),         // ignora si el valor no cambió
  switchMap(q => this.api.buscar(q)),  // cancela la búsqueda anterior
  takeUntilDestroyed(this.destroyRef)  // evita memory leaks
).subscribe(res => this.resultados.set(res));
```

**Operadores de aplanado — pregunta trampa frecuente:**

| Operador | Comportamiento | Caso de uso |
|---|---|---|
| `switchMap` | Cancela la anterior | Búsqueda / autocomplete |
| `mergeMap` | Ejecuta todas en paralelo | Subidas independientes |
| `concatMap` | Encola y respeta orden | Guardados secuenciales |
| `exhaustMap` | Ignora nuevas mientras haya una activa | Botón "login" anti doble-click |

**Memory leaks:** toda suscripción manual debe cerrarse (`takeUntil`, `takeUntilDestroyed`) o evitarse usando el pipe `async` en el template.

## 2.5 Formularios

```typescript
// Reactive Forms (recomendado para lógica real)
this.form = this.fb.group({
  email: ['', [Validators.required, Validators.email]],
  clave: ['', [Validators.required, Validators.minLength(8)]]
});

if (this.form.invalid) { this.form.markAllAsTouched(); return; }
this.api.login(this.form.getRawValue()).subscribe(...);
```

- **Template-driven** (`ngModel`): formularios simples, poca lógica.
- **Reactive**: validación compleja, testeable, tipado. **Es el que debes defender.**

## 2.6 HTTP + Interceptor (aquí se conecta con JWT) 🎯

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(TokenService).get();
  const authReq = token
    ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } })
    : req;

  return next(authReq).pipe(
    catchError((err: HttpErrorResponse) => {
      if (err.status === 401) inject(AuthService).logout();
      return throwError(() => err);
    })
  );
};
```

## 2.7 Routing, lazy loading y guards

```typescript
export const routes: Routes = [
  { path: '', component: HomeComponent },
  {
    path: 'admin',
    canActivate: [authGuard],
    loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
  },
  { path: '**', component: NotFoundComponent }
];

export const authGuard: CanActivateFn = () =>
  inject(AuthService).estaAutenticado() || inject(Router).createUrlTree(['/login']);
```

**Lazy loading** = cada módulo/ruta se descarga solo cuando se navega → bundle inicial más chico → mejor tiempo de carga.

## 2.8 Change detection 🎯

- **Default:** Angular revisa todo el árbol ante cualquier evento.
- **OnPush:** el componente solo se revisa si cambia una referencia de `@Input`, se emite un evento propio, o se dispara manualmente (`markForCheck`).
- **Signals:** reactividad granular, actualiza solo lo que depende de la señal. Es la dirección hacia donde va Angular (zoneless).

> Respuesta corta: *"Uso OnPush con objetos inmutables, o signals en proyectos nuevos, para que Angular no recorra todo el árbol en cada evento."*

## 2.9 Testing

```typescript
describe('UsuariosComponent', () => {
  beforeEach(() => TestBed.configureTestingModule({
    imports: [UsuariosComponent],
    providers: [provideHttpClientTesting()]
  }));

  it('cuenta usuarios activos', () => {
    const fixture = TestBed.createComponent(UsuariosComponent);
    fixture.componentInstance.usuarios.set([{id:1,activo:true},{id:2,activo:false}]);
    expect(fixture.componentInstance.totalActivos()).toBe(1);
  });
});
```
Jasmine + Karma (clásico), Jest (frecuente en empresas), Cypress/Playwright para E2E.

## 2.10 Preguntas típicas de Angular
- Diferencia entre `Observable` y `Promise` → *Observable es un stream, lazy, cancelable y multi-valor; Promise es un valor único, eager y no cancelable.*
- `switchMap` vs `mergeMap` → tabla de arriba.
- ¿Cómo evitas memory leaks?
- ¿Qué es la inyección de dependencias en Angular y qué es un `providedIn: 'root'`?
- ¿Cómo manejas el estado global? → Servicio con `BehaviorSubject`/signals para casos simples, NgRx si hay muchos eventos y trazabilidad.
- ¿Cómo aseguras rutas? → Guards + validación real en el backend.
