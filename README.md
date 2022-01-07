# First Person Shooter

## Inleiding

In dit project maken wij een First Person Shooter die nek aan nek met een agent targets zal moeten neerschieten in VR. Je zal zoveel mogelijk punten moeten verzamelen in een zo kort mogelijke tijd en zo dan meer punten te hebben dan de agent.

In de READ.ME file zal de lezer te weten komen wat je allemaal kunt doen in deze applicatie en hoe alles werkt. We zullen de resultaten van de agent training laten zien met daarbij een conclusie wat wij geleerd hebben en wat de conclusie juist inhoudt. In dit project zal je tussen verschillende moeilijkheidsgraad kunnen kiezen, je kan kiezen tussen 'easy', 'normal' en 'hard'. Hierna zal het spel beginnnen en heb je 60 seconden om zo veel mogelijk targets neer te schieten. Je zal punten krijgen voor elk target dat je neerschiet, om de 10 seconden zal er ook een priority target spawnen waar de user meer punten voor krijgt om neer te schieten. Er zullen ook ally targets spawnen waar je minpunten voor zal krijgen als je deze neerschiet. In de game is er ook een camera te zien van de agent waar je zijn gameplay zult kunnen volgen inclusief zijn score. Op het einde van de 60 seconden zal het spel stoppen en zul je jou score kunnen vergelijken met de score van de agent om te zien wie er gewonnen heeft, ten slotte zal er boven je een knop verschijnen waar je op kan schieten en het spel herstart wordt.

## Methoden
### Installatie
#### Unity
* Cinemachine - 2.6.11
* Input System - 1.2.0
* JetBrains Rider Editor - 2.0.7
* ML Agents - 2.0.0
* Oculus XR Plugin - 1.11.2
* OpenXR Plugin - 1.2.8
* Test Framework - 1.1.29
* TextMeshPro - 3.0.6
* Timeline - 1.4.8
* Unity UI - 1.0.0
* Universal RP - 10.7.0
* Version Control - 1.15.4
* Visual Studio Code Editor - 1.2.4
* Visual Studio Editor - 2.0.12
* XR Interaction Toolkit - 1.0.0-pre.3
* CR Plugin Management - 4.2.0

#### Extern
* Anaconda 3
* Ml-agents: 0.27.0 (Release 18)
* PyTorch: 1.10.0

### Spelverloop
Het spel start op, de gebruiker heeft een geweer in zijn rechterhand en 3 knoppen voor zich. De 3 knoppen dienen om de moeilijkheidsgraad te bepalen van het spel, de keuzens zijn respectievelijk: makkelijk (easy), normaal (normal) en moeilijk (hard). Het spel zal even laden en daarna begint het, je hebt 60 seconden de tijd om doelwitten te raken (rode kleding), elk doelwit is 1 punt waard. Wanneer je een bondgenoot (groende kleding) raakt verlies je 2 punten en wanneer je een prioriteit-doelwit raakt (blauwe kleding), verdien je 3 punten. Boven je is er een scherm waardoor de camera van de agent zichtbaar is, zo kan je ook de agent zijn score bekijken en deze vergelijken met je eige scoren in de rechter-bovenkant van je scherm. Je kan de agent ook zien als je naar de linkerzijde van de arena kijkt. Door de ondergrond en muur tussen agent en speler kan niet geschoten worden. Na deze 60 seconden is er een knop recht boven je waarmee je het spel opnieuw kan laten starten en een nieuwe moeilijkheidsgraad kan kiezen. Deze acties worden met bejulp van een GameManager script gedaan.

* GameManager.cs:
De *SetDifficulty(int difficulty)* methode wordt via DifficultySelector.cs opgeroepen. Deze methode zet de juiste agent actief en verbergt de knoppen (easy, normal, hard) van je scherm. Hierna wordt er een coroutine gestart waarin we aftellen tot het einde van het spel (60 seconden), dit staat in een for loop zodat we de timer ook kunen tonen op het scherm. Ook zorgen we er voor dat er in beiden de speler en agent omgevingen doelwitten komen. Wanneer het spel eindigt tonen we de eind-score van de gebruiker en stoppen we de environment van nieuwe doelwitten te laten verschijnen.
```{r}
    void Start()
    {
        scoreBoard.gameObject.SetActive(true);
        timer.gameObject.SetActive(true);
        gameOverCanvas.gameObject.SetActive(false);
        gameOverButton.gameObject.SetActive(false);

        easyAgent.gameObject.SetActive(false);
        normalAgent.gameObject.SetActive(false);
        hardAgent.gameObject.SetActive(false);
    }
    
    public void SetDifficulty(int difficulty)
    {
        switch (difficulty)
        {
            case 1:
                easyAgent.gameObject.SetActive(true);
                break;
            case 2:
                normalAgent.gameObject.SetActive(true);
                break;
            case 3:
                hardAgent.gameObject.SetActive(true);
                break;
        }
        easyButton.gameObject.SetActive(false);
        normalButton.gameObject.SetActive(false);
        hardButton.gameObject.SetActive(false);
        StartCoroutine(GameOver());
    }

    private IEnumerator GameOver()
    {
        switch (gameState)
        {
            case GameStates.Playing:
                spawner.enabled = true;
                spawnerAgent.enabled = true;
                spawner.ClearEnvironment();
                spawner.StartEnvironment();
                AgentEnvironment.StartEnvironment();


                for (int i = awaitTime; i > -1; i--)
                {
                    yield return new WaitForSeconds(1);
                    timer.text = i.ToString();
                }

                Debug.Log("einde spel");
                scoreBoard.gameObject.SetActive(false);
                timer.gameObject.SetActive(false);
                gameOverCanvas.gameObject.SetActive(true);
                gameOverCanvas.text = "End " + scoreBoard.text;
                spawner.ClearEnvironment();
                AgentEnvironment.ClearEnvironment();
                gameOverButton.gameObject.SetActive(true);
                
       
                gameState = GameStates.GameOver;
                break;
        }
    }
```

* DifficultySelector.cs: Wanneer de speler de knop met de tag Easy, Normal of Hard raakt stuurt deze de juiste moeilijkheidsgraad door naar de Game Manager.
```{r}
    private void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.tag == "BulletPlayer" && gameObject.tag == "Easy")
        {
            gameManager.SetDifficulty(1);
            Destroy(collision.gameObject);
        }
        else if (collision.gameObject.tag == "BulletPlayer" && gameObject.tag == "Normal")
        {
            gameManager.SetDifficulty(2);
            Destroy(collision.gameObject);

        }
        else if (collision.gameObject.tag == "BulletPlayer" && gameObject.tag == "Hard")
        {
            gameManager.SetDifficulty(3);
            Destroy(collision.gameObject);
        }
    }
```

* EnvironmentSpawner.cs: Hierin staat de logica om doelwitten in een omgeving te krijgen, een normaal doelwit wordt elke 0.5 tot 2 seconden toegevoegd. Een bondgenoot elke 10 tot 20 seconden en een prioritair doelwit elke 15 tot 25 seconden. Elke target heeft 4 mogelijkheiden, dit zijn de 4 quadranten van de arena. Oftewel positieve x en z, negatieve x en z, positieve x en negatieve z, negatieve x en positieve z. De targets zijn altijd tussen een afstand van 10 tot 40 verwijdert van de speler en agent. Wanneer een target in de arena terecht komt zal deze zich naar de speler richten. De GameManager roept de *StartEnvironment()* en *ClearEnvironment()* methoden op, *StartEnvironment()* start de spawners met behulp van co-routines en de *ClearEnvironment()* stopt deze co-routines en vernietigt alle doelwitten die nog bestaan.
```{r}
private void OnEnable()
    {
        target = transform.Find("Target").gameObject;
        ally = transform.Find("Ally").gameObject;
        priority = transform.Find("Priority").gameObject;
        StartCoroutine(Spawner());
        StartCoroutine(AllySpawner());
        StartCoroutine(PrioritySpawner());
    }

    private IEnumerator Spawner()
    {
        while (spawnTargets)
        {
            var waitTime = Random.Range(0.5f, 2f);
            yield return new WaitForSeconds(waitTime);
            SpawnTarget(target, targetPrefab);
        }
    }

    private IEnumerator AllySpawner()
    {
        while (spawnAllies)
        {
            var waitTime = Random.Range(10f, 20f);
            yield return new WaitForSeconds(waitTime);
            SpawnTarget(ally, allyPrefab);
        }
    }
    private IEnumerator PrioritySpawner()
    {
        while (spawnPriorities)
        {
            var waitTime = Random.Range(15f, 25f);
            yield return new WaitForSeconds(waitTime);
            SpawnTarget(priority, priorityPrefab);
        }
    }

    private void SpawnTarget(GameObject gameObject, GameObject gamePrefab)
    {
        var random = Random.Range(1, 5);
        float zRandom;
        float xRandom;
        switch (random)
        {
            case 1:
                zRandom = Random.Range(10f, 40f);
                xRandom = Random.Range(10f, 40f);
                break;
            case 2:
                zRandom = Random.Range(10f, 40f);
                xRandom = Random.Range(-10f, -40f);
                break;
            case 3:
                zRandom = Random.Range(-10f, -40f);
                xRandom = Random.Range(-10f, -40f);
                break;
            case 4:
                zRandom = Random.Range(-10f, -40f);
                xRandom = Random.Range(10f, 40f);
                break;
            default:
                zRandom = 0;
                xRandom = 0;
                break;
        }
        GameObject newTarget = Instantiate(gamePrefab);
        newTarget.transform.SetParent(gameObject.transform);
        newTarget.transform.localPosition = new Vector3(xRandom, 0, zRandom);
        newTarget.transform.LookAt(Vector3.zero);
    }

    public void StartEnvironment()
    {
        StartCoroutine(Spawner());
        StartCoroutine(AllySpawner());
        StartCoroutine(PrioritySpawner());
    }
    public void ClearEnvironment()
    {
        StopAllCoroutines();

        foreach (Transform target in target.transform)
        {
            GameObject.Destroy(target.gameObject);
        }
        foreach (Transform ally in ally.transform)
        {
            GameObject.Destroy(ally.gameObject);
        }
        foreach (Transform priority in priority.transform)
        {
            GameObject.Destroy(priority.gameObject);
        }   
    }
```

#### Observaties
Alle agents werken met ray perception sensors. 

* Easy-agent: De Easy-agent heeft 2 rays, één voor en één achter de agent. De rays hebben een lengte van 50 waardoor de agent niet alles kan zien in de arena. Deze rays kunnen alle objecten zien op de hoogte van y=0, met een sphere cast radius van 0.5.

* Normal-agent:  De Normal-agent heeft 10 ray perception sensors verdeeld over 360 graden zonder een sphere cast radius (om een acurater mikpunt te verkrijgen), deze hebben een lengte van 70 waardoor heel de arena zichtbaar is. Verder is er een verticale start en eind offset toegepast op de hoogt van y=2 zodat deze enkeld de hoofden van de targets kan zien. De normale-agent ziet enkel de objecten met de tags: Target, Ally en Priority.

*  Hard-agent: Deze agent wouden we eerst met een camera-sensor laten werken maar achteraf zijn we toch bij de ray perception sensor gebleven. De reden hiervoor is dat we ervoor zorgen dat de agent een target kan zoeken en hier direct naar kan kijken wanneer een ray deze raakt. Hierdoor moet de agent niet meer ronddraaien om een naar een doelwit te mikken. Zoals de normale-agent kan deze enkel objecten met de tags: Target, Ally en Priority zien. De rays hebben een lengte van 70 en er zijn 50 rays over 360 graden. Deze hebben een sphere cast radius van 1 en zijn ook in een offset van y=2.

#### Acties
Alle agents hebben 4 acties maar de hard-agent heeft er andere tegenoven de normal-, easy-agent. De schoten hebben een fire rate van 0.1 en een snelheid van 250.

##### Easy-, Normal-agent:
Deze agents kunnen links en rechts roteren alsook schieten. De methoden werken als volgt zoals getoont in *ShooterAgent.cs*.
* ShooterAgent.cs

*TurnLeft()*
```{r}
    private void TurnLeft()
    {
        var turnSpeed = Time.deltaTime * this.turnSpeed;
        transform.eulerAngles -= new Vector3(0, turnSpeed, 0);
    }
```
*TurnRight()*
```{r}
    private void TurnRight()
    {
        var turnSpeed = Time.deltaTime * this.turnSpeed;
        transform.eulerAngles += new Vector3(0, turnSpeed, 0);
    }
```
*Shoot()*
```{r}
    private void Shoot()
    {
        if (timer > fireRate)
        {
            GameObject newBullet = Instantiate(bullet, new Vector3(-180, 1, 0) + transform.forward, transform.rotation);
            newBullet.GetComponent<Rigidbody>().AddForce(transform.forward * bulletSpeed, ForceMode.VelocityChange);
            shoot = true;
            timer = 0f;
        }
        if (shoot)
        {
            if (timer < fireRate)
            {
                timer += Time.deltaTime;
            }
            else
            {
                timer = fireRate;

            }
        }
    }
```

#### Hard-agent
Het schieten blijft hetzelfde maar deze agent draait niet om rond zicht te kijken maar gebruikt de perception-sensors. De *FindTarget()* methode kijkt of er een ray is die een tagged object heeft geraakt (Ally, Priority of Target), zo ja kijkt of het object nog bestaat of niet. Indien deze voorwaarde voldaan zijn zal de locatie van dit object gebruikt worden en kijkt de agent naar deze positie. Ook kan de agent zijn target clearen met *ClearTarget()* dit betekent dat hij een nieuwe target kan zoeken met *FindTarget()*.

* ShooterAgentRay.cs

*FindTarget()*
```{r}
    public void FindTarget()
    {
        if (location == Vector3.zero)
        {
            foreach (var rayOutput in outputs.RayOutputs)
            {
                if (rayOutput.HitTaggedObject && rayOutput.HitGameObject != null)
                {
                    Debug.Log("Target found");
                    location = rayOutput.HitGameObject.transform.position;
                    transform.LookAt(location);
                }
            }
        }
    }
```

*ClearTarget()*
```{r}
    public void ClearTarget()
    {
        if (location != Vector3.zero)
        {
            location = Vector3.zero;
        }
    }
```

#### Beloningen
Er worden voor de agent 2 scores bij gehouden, deze zijn de beloningen voor de agent zelf en de score. Dit wordt berekent in het *OnHit.cs* script, de scores voor de agent en speler zijn hetzelfde. Verder is er ook een beloning wanneer een object na een bepaalde tijd verloopt, dit wordt later uitgelegd in [Objecten](#objecten).

* OnHit.cs

*OnCollisionEnter(Collision collision)*: Het object met tag *Bullet* is de agent, *BulletPlayer* is de speler. Wanneer de agent een gewoon doelwit raakt krijgt deze een beloning van 5 en een score van 1. Een prioritair doelwit is een beloning van 10 en score van 3 waard, en een bondgenoot heeft een beloning van -50 en een score van -2. Het uitdelen van beloningen en score gebeurt in *AddScore(float reward, int score)*. Voor de speler is er een methode in *ShooterPlayer.cs* genaamd *AddReward(float reward)* hier later meer over in [Objecten](#objecten).
```{r}
    private void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.tag == "Bullet" && gameObject.tag == "Target")
        {
            Debug.Log("Hit Enemy");
            AddScore(5f, 1);
            DestroyObject(collision, gameObject);
        }
        else if (collision.gameObject.tag == "Bullet" && gameObject.tag == "Priority")
        {
            Debug.Log("Hit Priority");
            AddScore(10f, 3);
            DestroyObject(collision, gameObject);
        }
        else if (collision.gameObject.tag == "Bullet" && gameObject.tag == "Ally")
        {
            Debug.Log("Hit Ally");
            AddScore(-50f, -2);
            DestroyObject(collision, gameObject);
        }
        else if (collision.gameObject.tag == "BulletPlayer" && gameObject.tag == "Target")
        {
            player.AddReward(1f);
            DestroyObject(collision, gameObject);
        }
        else if (collision.gameObject.tag == "BulletPlayer" && gameObject.tag == "Priority")
        {
            player.AddReward(3f);
            DestroyObject(collision, gameObject);
        }
        else if (collision.gameObject.tag == "BulletPlayer" && gameObject.tag == "Ally")
        {
            player.AddReward(-2f);
            DestroyObject(collision, gameObject);
        }
    }
```

*AddScore(float reward, int score)*: Hierin krijgen de agent(s) hun beloningen en score.
```{r}
    private void AddScore(float reward, int score)
    {
        if (agent != null)
        {
            agent.AddReward(reward);
            agent.agentScore += score;
        } else if (agentRay != null)
        {
            agentRay.AddReward(reward);
            agentRay.agentScore += score;
            agentRay.ClearTarget();
        }
    }
```

*OnActionReceived(ActionBuffers actions) - Easy, Normal Agent*: Wanneer de agent een actie kiest geven we hier ook een minimale beloning, zodat de agent weet dat hij dit (niet) vaker mag doen. 
```{r}
    public override void OnActionReceived(ActionBuffers actions)
    {
        var action = actions.DiscreteActions;
        if (action[0] == 1)
        {
            AddReward(0.000001f);
            TurnLeft();
        }
        else if (action[0] == 2)
        {
            AddReward(0.000001f);
            TurnRight();
        } 
        else if (action[0] == 3)
        {
            //AddReward(-0.05f);
            AddReward(-0.000001f);
            Shoot();
        }
    }
```

*OnActionReceived(ActionBuffers actions) - Hard Agent*: De hard agent wordt zwaarder gestraft bij het maken van acties.
```{r}
    public override void OnActionReceived(ActionBuffers actions)
    {

        var action = actions.DiscreteActions;
        if (action[0] == 1)
        {
            AddReward(-0.001f);
            FindTarget();
        }
        else if (action[0] == 2)
        {
            AddReward(-0.0001f);
            ClearTarget();
        }
        else if (action[0] == 3)
        {
            AddReward(-0.05f);
            Shoot();
        }
    }
```

#### Objecten
* Doelwitten: Hoe de doelwitten tevoorschijn komen werd uitgelegd in [Spelverloop](#spelverloop), de beloningen van de doelwitten werdt uitgelegd in [Beloningen](#beloningen). Een doelwit wordt ook vernieitigt nadat deze is geraakt, of na een bepaalde timer. De timer voor een normaal doelwit is 8seconden, bondgenoot is ook 8seconden en een prioritair doelwit is 10s. Nadat deze timer afloopt wordt het via *ObjectDestroyer.cs* vernietigt. Bij de prefab van een prioritair target is er ook een sound toegewezen zodat de speler kan horen wanneer deze tevoorschijn komt en ook waar met behulp van spatial blend en logarithmic rolloff.

*GameObjectDestroy()*: Deze methode wacht voor de tijd die werd meegegeven vanuit het object en vernietigt het object.
```{r}
    private IEnumerator GameObjectDestroy()
    {
        yield return new WaitForSeconds(lifeTime);
        despawned = true;
        Destroy(gameObject);
    }
```

*OnDestroy()*: Hierin wordt een beloning toegewezen aan de agent. Indien het de hard-agent is wordt er ook nagekeken of het doelwit dat verlopen is, gelijk staat aan het doelwit waar de agent momenteel naar kijkt. Als dit het geval is zal de agent dit weten doordat de *ClearTarget()* methode opgeroepen wordt. Indien een bondgenoot automatisch verloopt krijgt de agent een beloning van 1, als een doelwit of prioritair doelwit verloopt krijgt de agent een negatieve beloning van 1.
```{r}
    private void OnDestroy()
    {
        if (agentRay != null)
        {
            if (gameObject.transform.position.Equals(agentRay.Location))
            {
                agentRay.ClearTarget();
            }
        }
        if (despawned && agent != null)
        {
            switch (gameObject.tag)
            {
                case "Ally":
                    agent.AddReward(1f);
                    break;
                case "Priorty":
                    agent.AddReward(-1f);
                    break;
                case "Target":
                    agent.AddReward(-1f);
                    break;
                default:
                    break;
            }
        }
        else if (despawned && agentRay != null)
        {
            switch (gameObject.tag)
            {
                case "Ally":
                    agentRay.AddReward(1f);
                    break;
                case "Priorty":
                    agentRay.AddReward(-1f);
                    break;
                case "Target":
                    agentRay.AddReward(-1f);
                    break;
                default:
                    break;
            }
        }
        despawned = false;
    }
```

* Speler: De speler werkt enkel met een RightHandController waarin de XR-componenten staan, verder is er een camera en een scherm waarin de camera van de agent te zien is. De schiet methode van de speler is hetzelfde als die van de agents, enkel de *AddReward(float reward)* methode voegt de score toe aan een variabele zodat we deze kunnen bijhouden en maken we gebruik van een **UnityEvent** en **EventWatcher** dit gebeurt in *PrimaryButtonWatcher.cs*

*PrimaryButtonWatcher.cs - PrimaryButtonEvent*: hierin maken we het event aan, aangezien dit over de primary button ("a" op de controller) gaat kan deze true of false zijn -> bool.
```{r}
[System.Serializable]
public class PrimaryButtonEvent : UnityEvent<bool> { }
```

*Awake()*: wanneer het event nog niet bestaat maken we hier een nieuwe instantie van. Ook initialiseren we een nieuwe lijst waarin later alle mogelijke input devices (Oculus Quest in dit geval) terecht komen met een primary button ("a").
```{r}
public class PrimaryButtonWatcher : MonoBehaviour
{
    public PrimaryButtonEvent primaryButtonDown;
    private bool previousButtonState = false;
    private List<InputDevice> devicesPrimaryButton;

    private void Awake()
    {
        if (primaryButtonDown == null)
        {
            primaryButtonDown = new PrimaryButtonEvent();
        }


        devicesPrimaryButton = new List<InputDevice>();
    }
}
```

*OnEnable()*: Hierin maken we een nieuwe lijst waarin alle beschikbare apparaten in komen te staan, als deze een primary button hebben ("a") en momenteel beschikbaar zijn (geconnecteerd) dan voegen we het apparaat toe aan de lijst met apparaten die een primary button hebben via *InputDevices_Connected(InputDevice device)*. Indien het apparaat niet meer geconnecteerd is dan halen we deze uit de lijst van beschikbare apparaten via *InputDevices_Disconnected(InputDevice device)*.
```{r}
    private void OnEnable()
    {

        List<InputDevice> allDevices = new List<InputDevice>();
        InputDevices.GetDevices(allDevices);
        foreach (InputDevice device in allDevices)
        {
            InputDevices_Connected(device);
        }

        InputDevices.deviceConnected += InputDevices_Connected;
        InputDevices.deviceDisconnected += InputDevices_Disconnected;
    }

    void InputDevices_Connected(InputDevice device)
    {
        bool discardedValue;
        if (device.TryGetFeatureValue(CommonUsages.primaryButton, out discardedValue))
        {
            devicesPrimaryButton.Add(device);
        }
    }

    void InputDevices_Disconnected(InputDevice device)
    {
        if (devicesPrimaryButton.Contains(device))
        {
            devicesPrimaryButton.Remove(device);
        }
    }
```

*OnDisable()*: Bij het uitzetten van het spel zullen alle lijsten leeg worden gemaakt. 
```{r}
    private void OnDisable()
    {
        InputDevices.deviceConnected -= InputDevices_Connected;
        InputDevices.deviceDisconnected -= InputDevices_Disconnected;
        devicesPrimaryButton.Clear();
    }
```

*Update()*: Hierin halen we de verbonden apparaten op, indien de primary button ("A") ingedrukt is wordt dit in de *primaryButtonState gezet als true*. We kijken ook na of deze knop al ingedrukt was, indien dit het geval is gebeurt er niets met de tempstate aangezien het op true moet blijven staan. Als dat niet het geval is dan wordt er gekeken of de *primaryButtonState* nu wel true is, zo ja wordt de tempstate op true gezet. Hierdoor weet *ShooterPlayer.cs* dat de knop ingeduwt is door *onPrimaryButtonEvent(bool pressed)* aangezien deze opgeroepen wordt met behulp van *primaryButtonDown.Invoke(tempstate)* indien de status van de knop wijzigt.
```{r}
    private void Update()
    {
        bool tempstate = false;
        foreach (var device in devicesPrimaryButton)
        {
            bool primaryButtonState = false;
            tempstate = device.TryGetFeatureValue(CommonUsages.primaryButton, out primaryButtonState) && primaryButtonState || tempstate;
        }

        if (tempstate != previousButtonState)
        {
            primaryButtonDown.Invoke(tempstate);
            previousButtonState = tempstate;
        }
    }
```

*ShooterPlayer.cs - Shoot when primary button pressed*
```{r}
    [SerializeField]
    private PrimaryButtonWatcher watcher;
    public bool isPressed = false;
    private Coroutine shootRoutine;

    void Start()
    {
        transform.localPosition = new Vector3(0, 0, 0);

        watcher.primaryButtonDown.AddListener(onPrimaryButtonEvent);
    }

    void onPrimaryButtonEvent(bool pressed)
    {
        isPressed = pressed;
        //Shoot();
        if (shootRoutine != null)
        {
            StopCoroutine(Shoot());
        }
        shootRoutine = StartCoroutine(Shoot());
    }
```

#### One-pager
* Afwijkingen: De enige afwijking is dat de muur waardoor de speler de agent kan zien niet rechts van de speler staat maar links. Verder zijn de moeilijkheidsgraden vooral op snelheid gericht, in *easy* zal de agent bondgenoten raken en soms vast lopen op een target totdat deze verdwijnt. In *normal* zal de agent minder vaak bondgenoten raken en accurater schieten, in *hard* gaat de agent abnormaal snel en heeft deze een erg lage kans om bondgenoten te raken, ongeveer 9% (1/11) kans wanneer een bondgenoot in zicht is. Tenslotte krijgen de speler en agent geen bonuspunten voor hoofschoten (dit was een mogelijke uitbereiding in de one-pager).
De one-pager zelf vindt je terug [hier](./One-Pager.pdf)

## Resultaten

## Conclusie

Bij deze dan de conclusie van onze First Person Shooter, wat we hebben geleerd met het maken van een VR project en het trainen van een agent om hetzelfde te doen simultaan met de speler. 

We hebben drie verschillende moeilijkheidsgraden getrained, 'easy', 'normal' en 'hard'. De eerste die we hebben getraind is 'easy' en deze hebben we voor een lange tijd laten draaien. Hierna hebben we 'normal' en 'hard' getrained maar deze hebben we minder lang laten draai omdat we doorhadden dat dit helemaal niet nodig was. We zien dat de training na een bepaalde tijd stabiliseerd en dat de agent getrained is.

We hadden toch door na dat we begonnen waren dat de training meer moeite ging kosten dan gedacht want we moesten de agent eerst apart kunnen laten schieten en daarna voegde we telkens acties toe. Na het schieten voegde we verschillende targets toe zodat de agent wist op welk target hij wel moest schieten en welk target niet en hierna voegde we ook toe dat de agent kon draaien en de target kon zoeken. Dit was niet allemaal even evident dus hebben we de agent de eerste paar stappen met heuristic aangeleerd.

In de toekomst zullen we misschien de taken van de agent opsplitsen zodat de agent alle taken in kleine hapjes kan leren en niet alles tegelijkertijd, dit zal ons tijd besparen bij de trainingen. We hadden ook problemen bij de input van de Oculus te lezen en na veel tijd hebben we toch een complexe oplossing gevonden dus deze kunnen we ook toepassen bij toekomstige projecten.
