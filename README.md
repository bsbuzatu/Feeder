# Feeder
Feeding app
# Jurnal Nutrițional - Bogdan

Aplicație web personalizată pentru tracking-ul zilnic al nutriției, măsurătorilor corporale și evoluției compoziției corporale.

## Caracteristici

- **Tracking macronutrienți** cu ținte personalizate (proteine 2g/kg, calorii deficit, carbohidrați auto-calculate)
- **Alertă specială pentru fier** (limită 10mg/zi pentru profilul tău cu HFE + feritină crescută)
- **Bază de date 60+ alimente** din planul nutrițional aprobat
- **Măsurători zilnice**: greutate, talie, % grăsime, pași, somn, antrenament
- **Grafice de evoluție** pentru greutate, talie, proteine, calorii
- **Export/Import JSON** pentru analiză periodică
- **Salvare locală** (IndexedDB) - datele rămân pe telefonul tău, nu se trimit nicăieri
- **Mobile-first**, optimizat pentru iPhone/Android
- **Funcționează offline** după prima încărcare (poate fi adăugat pe Home Screen ca "app")

## Țintele tale (calculate automat din 97.4 kg)

| Macronutrient | Țintă | Calcul |
|---|---|---|
| **Calorii** | 2.000 kcal | Deficit ~500 kcal vs TDEE |
| **Proteine** | 195 g | 2.0 g/kg corporal |
| **Carbohidrați** | 190 g | Restul caloriilor după proteine + grăsimi |
| **Grăsimi** | 80 g | ~36% din calorii (suport hormonal) |
| **Fibre** | 35 g | Minim pentru sănătate metabolică |
| **Fier MAX** | 10 mg | Limită strictă (HFE + feritină 368) |

Toate valorile pot fi modificate din tab-ul **Setări**.
