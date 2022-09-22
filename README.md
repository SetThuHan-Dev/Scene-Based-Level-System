# Scene-Based-Level-System
Re-useable codes for saved level system when levels come with many scenes

### Purpose: for someone who wants to Destroy all gameObjects and load a new scene by saving and reloading variables in Json or PlayerPref

> This LevelManager mustn't be DontDestroyOnLoad and each scene must include this LevelManager
> Scene 0 as Entry
> And Code starts to load Scene 1
> Saved Current Level on Application Quit

### Preview -- In Scene
![Ss](https://user-images.githubusercontent.com/113447169/191726112-3eb9a82b-6dba-4f58-a145-011a00047d33.png)


```csharp
using UnityEngine;
using Sabresaurus.PlayerPrefsUtilities;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class LevelManager : MonoBehaviour
{
    public static LevelManager Instance;

    public Text levelText;
    public LifeCount lifeCount;

    private int Level;
    private int RandomLevel;
    private int RandomLevelIndex;
    private int MinLevel = 1;
    private int MaxLevel = 10;

    private bool IsLevelLoaded;
    private bool IsGenerated;

    private void Awake()
    {
        Instance = this;

        if (!PlayerPrefsUtility.HasEncryptedKey("LevelLoaded"))
        {
            IsLevelLoaded = true;
            PlayerPrefsUtility.SetEncryptedBool("LevelLoaded", IsLevelLoaded);
        }
        else
        {
            IsLevelLoaded = PlayerPrefsUtility.GetEncryptedBool("LevelLoaded");
        }
      
    }

    private void Start()
    {

        if (!PlayerPrefsUtility.HasEncryptedKey("LEVEL"))
        {
            Level = 1;
            levelText.text = "LEVEL " + Level;
            PlayerPrefsUtility.SetEncryptedInt("LEVEL", Level);
            lifeCount.CreateLive();   //
            SceneManager.LoadScene(Level);
 
        }

        else
        {
            OnStartLoadLevel();
        }

    }
    public void OnStartLoadLevel()
    {
     
        Level = PlayerPrefsUtility.GetEncryptedInt("LEVEL");
        levelText.text = "LEVEL " + Level;

        // example external function in which GameObjects are not DontDestroyOnLoad
        lifeCount.UpdateLive();

        if (!IsLevelLoaded)
        {
            // example external function in which GameObjects are not DontDestroyOnLoad
            lifeCount.CheckLive();

            if (Level > MaxLevel)
            {

                IsGenerated = PlayerPrefsUtility.GetEncryptedBool("IsGenerated");

                if (!IsGenerated)
                {
                    GenerateRandomLevelNumber();
                }

                RandomLevelIndex = PlayerPrefsUtility.GetEncryptedInt("RandomLevel");
                Level = RandomLevelIndex;
                levelText.text = "LEVEL " + Level;

                PlayerPrefsUtility.SetEncryptedBool("LevelLoaded", true);

                SceneManager.LoadScene(Level);

            }
            else
            {

                PlayerPrefsUtility.SetEncryptedBool("LevelLoaded", true);
                SceneManager.LoadScene(Level);

            }

        }

    }

    public void LoadNextLevel()
    {
  
        PlayerPrefsUtility.SetEncryptedBool("LevelLoaded", false);

        if (Level <= MaxLevel)
        {
            Level += 1;
            PlayerPrefsUtility.SetEncryptedInt("LEVEL", Level);
            SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
        }
        else
        {
            Level += 1;
            PlayerPrefsUtility.SetEncryptedInt("LEVEL", Level);
            PlayerPrefsUtility.SetEncryptedBool("IsGenerated", false);
            SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
        }

    }

    public void ReplayCurrentLevel()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }

    public void SkipCurrentLevel()
    {

        if (Level <= MaxLevel)
        {
            Level += 1;
            PlayerPrefsUtility.SetEncryptedInt("LEVEL", Level);
            SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
        }
        else
        {
            Level += 1;
            PlayerPrefsUtility.SetEncryptedInt("LEVEL", Level);
            PlayerPrefsUtility.SetEncryptedBool("IsGenerated", false);
            SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
        }

        PlayerPrefsUtility.SetEncryptedBool("LevelLoaded", false);

    }

    public int GenerateRandomLevelNumber()
    {
        PlayerPrefsUtility.SetEncryptedBool("IsGenerated", true);
        RandomLevel = Random.Range(MinLevel, MaxLevel);
        PlayerPrefsUtility.SetEncryptedInt("RandomLevel", RandomLevel);
        return RandomLevel;
    }

    private void OnApplicationQuit()
    {
        PlayerPrefsUtility.SetEncryptedBool("LevelLoaded", false);
    }

    public void GetLife()
    {
        lifeCount.SetDefaultAndShowReward();
    }
    
}

/// Example Class which is not DontDestroyOnLoad

[System.Serializable]
public class LifeCount
{
    [SerializeField] private GameObject LivePanel;
    [SerializeField] private Text text;
    [SerializeField] private int liveCount;

    public void CreateLive()
    {

        int count = 10;
        text.text = count.ToString("0");
        Debug.Log(text.text);
        PlayerPrefsUtility.SetEncryptedInt("Life", count);

    }

    public void CheckLive()
    {

        liveCount = PlayerPrefsUtility.GetEncryptedInt("Life");

        if (liveCount > 0)
        {
            liveCount--;
            text.text = liveCount.ToString("0");
            PlayerPrefsUtility.SetEncryptedInt("Life", liveCount);
            LivePanel.SetActive(false);

        }
        if (liveCount < 1)
        {
            text.text = liveCount.ToString("0");
            LivePanel.SetActive(true);
        }

    }

    public void UpdateLive()
    {

        liveCount = PlayerPrefsUtility.GetEncryptedInt("Life");
        text.text = liveCount.ToString("0");

        if (liveCount > 0)
        {
            LivePanel.SetActive(false);
        }

        if (liveCount < 1)
        {
            LivePanel.SetActive(true);
        }

    }

    public void SetDefaultAndShowReward()
    {

        liveCount = 10;
        text.text = liveCount.ToString("0");
        PlayerPrefsUtility.SetEncryptedInt("Life", liveCount);
        LivePanel.SetActive(false);

    }

}
```
