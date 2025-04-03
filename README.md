# HiraruCaseStudy Oyun Sistemi Dokümantasyonu

## İçindekiler
- [Gameplay Ability System](#gameplay-ability-system)
- [Karakter Sistemi](#karakter-sistemi)
- [Etkileşim Sistemi](#etkileşim-sistemi)
- [Silah Sistemi](#silah-sistemi)
- [Item Sistemi](#item-sistemi)

## Gameplay Ability System

### BaseGameplayAbility

`BaseGameplayAbility` sınıfı, tüm oyun yeteneklerinin temelini oluşturur ve Unreal Engine'in Gameplay Ability System (GAS) altyapısını kullanır.

```cpp
UCLASS()
class HIRARUCASESTUDY_API UBaseGameplayAbility : public UGameplayAbility
{
    // Class implementasyonu...
}
```

#### Önemli Özellikler:

- **EAbilityInputID**: Yeteneklerin hangi girişle tetiklendiğini belirlemek için kullanılan enum sınıfı
- **bActivateAbilityOnGranted**: Yetenek karaktere verildiğinde otomatik olarak aktifleştirilip aktifleştirilmeyeceğini belirler
- **AbilityInputID**: Yeteneğin hangi giriş tuşuyla tetikleneceğini belirler

#### Ana Metodlar:

- **OnAvatarSet**: Yetenek bir karaktere atandığında çağrılır

## Karakter Sistemi

### PlayerCharacter

`PlayerCharacter` sınıfı, oyuncunun kontrol ettiği ana karakteri temsil eder ve GAS entegrasyonu sağlar.

```cpp
UCLASS()
class HIRARUCASESTUDY_API APlayerCharacter : public AHiraruCaseStudyCharacter, public IAbilitySystemInterface
{
    // Class implementasyonu...
}
```

#### Önemli Özellikler:

- **AbilitySystemComponent**: Karakterin yetenek sistemini yöneten bileşen
- **DefaultAbilities**: Karaktere otomatik verilen yeteneklerin listesi
- **GrenadeCount**: Karakterin sahip olduğu el bombası sayısı
- **FireAction, GrenadeAction, HookAction, DashAction, InteractAction**: Girdi eylemleri

#### Ana Metodlar:

- **GetAbilitySystemComponent**: GAS bileşenine erişim sağlar
- **InitializeAbilities**: Varsayılan yetenekleri karaktere verir
- **SendAbilityLocalInput**: Girdi eylemlerini yetenek sistemine iletir
- **Interact**: Çevredeki Interactable interface kullanan nesnelerle etkileşime geçer
- **GetCooldownRemainingForTag**: Belirli bir yeteneğin kalan bekleme süresini döndürür

## Etkileşim Sistemi

### Interactable Arayüzü

`IInteractable` arayüzü, oyuncunun etkileşime geçebileceği tüm nesnelerin uygulaması gereken temel arayüzdür.

```cpp
UINTERFACE(MinimalAPI)
class UInteractable : public UInterface
{
    // Interface tanımı...
}
class HIRARUCASESTUDY_API IInteractable  
{  
public:  
    virtual void Interact(class APlayerCharacter* InteractingPlayer) = 0;  
    virtual FText GetInteractDisplayName() = 0;  
};
```

#### Ana Metodlar:

- **Interact**: Oyuncu nesneyle etkileşime geçtiğinde çağrılır
- **GetInteractDisplayName**: Etkileşim sırasında gösterilecek nesne adını döndürür

### Etkileşim Mekanizması

Oyuncu karakteri `InteractAction` girdi eylemi tetiklendiğinde, kamera yönünde bir ray trace gerçekleştirir. Eğer bu ray trace bir `IInteractable` arayüzünü uygulayan bir nesneye çarparsa, nesnenin `Interact` metodu çağrılır.

## Item Sistemi

### ItemBase

`ItemBase` sınıfı, oyunda toplanabilir tüm eşyaların temel sınıfıdır ve `IInteractable` arayüzünü uygular.

```cpp
UCLASS(Abstract)
class HIRARUCASESTUDY_API AItemBase : public AActor, public IInteractable
{
    // Class implementasyonu...
}
```

#### Önemli Özellikler:

- **Name**: Eşyanın görünen adı
- **Description**: Eşya açıklaması
- **Thumbnail**: Eşya görseli
- **Quantity**: Eşya miktarı
- **MaxStackSize**: Maksimum istiflenebilir miktar
- **bCanBeUsed**: Eşyanın kullanılabilir olup olmadığı

#### Ana Metodlar:

- **Interact**: Oyuncu eşyayla etkileşime geçtiğinde çağrılır
- **Pickup**: Eşya toplanırken işlenen mantık
- **OnItemPickup**: Blueprint tarafında uygulanabilen, eşya toplandığında çağrılan olay

## Silah Sistemi

### PSBulletBase

`PSBulletBase` sınıfı, silahlarda kullanılan mermilerin davranışını belirler.

```cpp
UCLASS()
class HIRARUCASESTUDY_API APSBulletBase : public AActor
{
    // Class implementasyonu...
}
```

#### Önemli Özellikler:

- **BulletTrailComponent**: Mermi izini gösteren Niagara bileşeni
- **ImpactEffect**: Çarpma efekti
- **DamageTypeToApply**: Uygulanacak hasar tipi (Radial, Point, PointWithPush)
- **BulletDamage**: Temel mermi hasarı
- **InitialVelocity**: Başlangıç hızı
- **GravityScale**: Yerçekimi etkisi çarpanı

#### Ana Metodlar:

- **Tick**: Her karede mermi fiziğini günceller
- **GetSimulationTimeStep**: Simulasyon adım zamanını hesaplar
- **PlayImpactEffects**: Çarpma efektlerini oynatır
- **GetTotalDamage**: Hasar değerini yüzey tipine göre hesaplar

#### Hasar Türleri (EDamageTypeCustom):

- **RadialDamage**: Çarpma noktasında alan hasarı uygular
- **PointDamage**: Sadece çarpılan hedefe hasar verir
- **PointDamageWithPush**: Çarpılan hedefe hasar verir ve itme kuvveti uygular

## Örnek Kullanım Akışları

### Yetenek Kullanımı

1. Oyuncu bir tuşa basar (`FireAction`, `GrenadeAction` vb.)
2. `SendAbilityLocalInput` metodu çağrılır
3. `AbilitySystemComponent` ilgili yeteneği etkinleştirir

### Nesne Etkileşimi

1. Oyuncu `InteractAction` tuşuna basar
2. Karakter `Interact` metodunu çağırır
3. Eğer oyuncu client ise `ServerInteract` metodu çağırılır.
4. Kamera yönünde bir ray trace yapılır
5. Ray trace bir `IInteractable` nesnesine çarparsa, nesnenin `Interact` metodu çağrılır
6. Nesne tipine göre uygun davranış gerçekleştirilir (örn. `ItemBase` için `Pickup` metodu)

### Mermi Fiziği

1. Silah ateşlendiğinde bir `PSBulletBase` örneği oluşturulur
2. Her karede `Tick` metodu ile mermi pozisyonu güncellenir
3. Mermi bir nesneye çarptığında `PlayImpactEffects` çağrılır
4. Hasar tipi ve yüzey tipine göre uygun hasar uygulanır
