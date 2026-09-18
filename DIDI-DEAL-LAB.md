# DiDi Deal Lab

Simulador de negociaciones para DiDi Food. Un solo archivo estático
(`didi-deal-lab.html`) — sin build, sin dependencias. Ábrelo en el navegador.

## Qué hace

1. **Puerta de entrada**: eliges si modelas **a nivel marca** (P&L blended, ~14
   variables) o **a nivel tienda** (mix delivery / pick-up, parque de tiendas,
   aperturas con ramp-up, inversión del deal).
2. **Baseline** — unit economics por orden: reparto del GMV, bridge del Take
   Rate Income al CM, P&L completo (por orden y total del periodo), capa de
   volumen y tornado de sensibilidad.
3. **Escenarios** — botón de crear escenario, seleccionas qué variables
   impactar, las editas con impacto en vivo, y queda guardado con sus deltas.
   Un escenario guarda **solo** los overrides: si cambia el baseline, todos se
   recalculan.
4. **Comparativo** — barras por KPI, frontera TED vs CM, bridge de atribución
   por escenario (de dónde viene cada peso del delta) y P&L comparativo línea
   por línea con chips de delta. Exporta a CSV.
5. **Proyecciones** — horizonte, ramp-up del deal, crecimiento mensual,
   aperturas con curva de productividad, y payback de la inversión del deal.

El estado se guarda en `localStorage`.

## El modelo

Todo el unit economics está en **MXN por orden**.

```
GMV             = AOP + Service Fee + Delivery Fee
R Burn          = %GMV x GMV
Take Rate Inc.  = (AOP - R Burn) x Contract Take Rate
Effective TR    = TRI / AOP
Business Income = TRI + Delivery Income + Service Fee
Business Costs  = Delivery Cost + D Burn + B2C + P2C + Meal Loss + Otros
CM sin Ads      = Business Income - Business Costs
CM con Ads      = CM sin Ads + Ads Revenue
TED             = R Burn + B2C + P2C
Brand's Income  = AOP - R Burn - TRI
```

### R Burn no es un costo

Es la regla que más se rompe al modelar esto. El R Burn lo financia el
restaurante, no DiDi. No entra a Business Costs. Lo que sí hace es **recortar
la base sobre la que se cobra el take rate**: por eso un R Burn alto baja el
Take Rate Income y abre la brecha entre el take rate contractual y el efectivo.
En la app aparece marcado con ▲ en todos lados.

### Pick-up (solo nivel tienda)

El canal pick-up no lleva Delivery Fee, ni Delivery Cost, ni D Burn; su take
rate es propio y mucho menor, y el Meal Loss se multiplica por un factor
(0.50x por defecto). El P&L blended pondera ambos canales por su mix de
órdenes.

### Proyecciones

El unit economics se mantiene constante en el horizonte; lo único que se
proyecta es el volumen (ramp-up del deal, crecimiento mensual y, en nivel
tienda, aperturas con su propia curva de productividad). Si esperas que el
take rate o los burns cambien en el tiempo, eso se modela como un escenario
aparte — mezclar las dos cosas hace imposible atribuir el resultado.

## Baselines de arranque

- **Nivel marca**: KA de café MX (AOP 324, service fee 17, delivery fee 21,
  contract TR 9.5%, R Burn 16% del GMV). Reproduce el P&L YTD de referencia
  dentro del redondeo: GMV 362, TRI 25.3, eff. TR 7.8%, TED 80.2,
  Brand's Income 240.8.
- **Nivel tienda**: cadena de 60 sedes, 8.9 órdenes/sede/día, 1% pick-up,
  contract TR 13%, R Burn 5% del GMV.

Ambos son puntos de partida editables, no verdades: cambia el baseline a la
realidad de tu cuenta antes de simular.
