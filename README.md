using UnityEngine;

public class AirplaneFlight : MonoBehaviour
{
    [Header("物件參考")]
    public Transform building; // 建築物 Transform

    [Header("音效設定")]
    public AudioSource flightAudioSource; // 播放飛行音效的 AudioSource
    public AudioClip flyingSound;          // 飛行音效
    public AudioClip crashSound;           // 撞擊音效
    public AudioClip passSound;            // 通關音效

    [Header("飛行參數設定")]
    public float baseForwardSpeed = 15f;   // 基本前進速度
    private float currentSpeed;            
    private float minHeight = 20f;          
    private float maxHeight = 50f;         
    
    [Header("起伏與高度調整")]
    public float waveAmplitude = 50f;      // 上下起伏幅度
    public float waveFrequency = 0.4f;     // 起伏頻率
    public float heightOffset = 15f;       // 【新增】整體高度向上平移的數值（數值越大，整條飛行路線越高）

    private float randomPhase;             // 隨機相位偏移
    private float randomOffsetSeed;        // 隨機雜訊種子

    private Vector3 initialAirplanePos = new Vector3(0f, 30f, 300f);
    private Vector3 initialBuildingPos = new Vector3(0f, 0f, 0f);

    private bool isFinished = false;       // 是否完成飛行
    private bool hasPassedBuilding = false; // 是否已飛越建築物

    private Rigidbody rb;                  // 宣告 Rigidbody 變數

    void Start()
    {
        rb = GetComponent<Rigidbody>();
        if (rb != null)
        {
            rb.useGravity = false;         
            rb.isKinematic = false;        
        }

        // 隨機化速度與相位，確保每次路線不同
        Random.InitState((int)System.DateTime.Now.Ticks);
        randomPhase = Random.Range(0f, 100f);
        randomOffsetSeed = Random.Range(0f, 50f);
        currentSpeed = baseForwardSpeed + Random.Range(-3f, 5f);

        // 自動計算建築物高度範圍
        if (building != null)
        {
            building.position = initialBuildingPos;
            Collider buildingCollider = building.GetComponent<Collider>();
            if (buildingCollider != null)
            {
                minHeight = buildingCollider.bounds.min.y;
                maxHeight = buildingCollider.bounds.max.y;
            }
            else
            {
                minHeight = 0f;
                maxHeight = 25f;
            }
        }

        transform.position = initialAirplanePos;

        if (flightAudioSource != null && flyingSound != null)
        {
            flightAudioSource.clip = flyingSound;
            flightAudioSource.loop = true;
            flightAudioSource.Play();
        }
    }

    void FixedUpdate() 
    {
        if (isFinished) return;

        Vector3 direction = (initialBuildingPos - initialAirplanePos).normalized;
        direction.y = 0; 

        // 計算高低起伏
        float timeVal = Time.time * waveFrequency + randomPhase;
        float wave = Mathf.Sin(timeVal) * waveAmplitude;
        float noise = (Mathf.PerlinNoise(Time.time * 0.3f, randomOffsetSeed) - 0.5f) * 5f;
        
        float centerHeight = minHeight + (maxHeight - minHeight) * 0.6f;
        
        // 【修改這裡】加上 heightOffset 讓整條軌跡向上平移
        float newY = centerHeight + wave + noise + heightOffset;

        if (newY < minHeight)
        {
            newY = minHeight;
        }

        Vector3 currentPos = rb != null ? rb.position : transform.position;
        currentPos += direction * currentSpeed * Time.fixedDeltaTime;
        currentPos.y = newY;

        if (rb != null)
        {
            rb.MovePosition(currentPos);
        }
        else
        {
            transform.position = currentPos;
        }

        // 判斷是否已飛越建築物
        if (!hasPassedBuilding && building != null && transform.position.z < building.position.z - 10f)
        {
            OnPassBuilding();
        }
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (isFinished) return;

        if (collision.transform == building || collision.gameObject.CompareTag("Building"))
        {
            OnCrashBuilding();
        }
    }

    void OnCrashBuilding()
    {
        isFinished = true;
        Debug.Log("飛機撞擊建築物！");

        if (flightAudioSource != null)
        {
            flightAudioSource.Stop();
            if (crashSound != null)
            {
                flightAudioSource.PlayOneShot(crashSound);
            }
        }
    }

    void OnPassBuilding()
    {
        hasPassedBuilding = true;
        isFinished = true;
        Debug.Log("順利飛越建築物！");

        if (flightAudioSource != null)
        {
            flightAudioSource.Stop();
            if (passSound != null)
            {
                flightAudioSource.PlayOneShot(passSound);
            }
        }
    }
}



1.  

2. https://github.com/toby011000006-ai/1151-HW4-414262375-CPC 

3. https://youtu.be/HfIUSv97dQk 



4.從網路上下載飛機及建築物(障礙物)3D模型，位兩者物件附加碰撞箱，編寫腳本:設定兩者初始位置，並使飛機飛向建築物，飛行途中隨機高低起伏(利用 Random Seed)並撥放飛行音效，若飛機碰撞到大樓則撥放爆炸音效並停止，若成功飛越則撥放成功音效。 

 
