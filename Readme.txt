点击singleplayer进行单人游戏，
点击multiplayer进行双人游戏，
游戏结束后点击Retry重新游戏，
点击Quit以退出游戏

脚本：
using System.Collections;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class GameController : MonoBehaviour
{
    [Header("PlayLogics")]
    public Button[] buttons; // 9个按钮
    public bool isSinglePlayer = true; // 是否为单人模式
    private string currentPlayer = "X"; // 当前玩家
    private int moveCount = 0; // 记录走的步数

    [Header("UI TimerSet")]
    public float sloganTime;
    public float BGTime;
    public float TODO;

    [Header("UI References")]
    public Text winnerAnouncement;
    public Canvas canvas;
    public GameObject grid;
    public Image bg;
    public Image slogan;
    public Image playGround;

    public Button[] startButtons;

    public Button retry;


    void Start()
    {
        for (int i = 0; i < buttons.Length; i++)
        {
            int index = i;
            buttons[i].onClick.AddListener(() => OnButtonClick(index));
            Debug.Log("Button " + index + " listener added.");
        }

    }
    public void OnButtonClick(int index)
    {
        if (moveCount < 9)
        {
            Debug.Log("Button clicked at index: " + index);


            Text buttonText = buttons[index].GetComponentInChildren<Text>();

            if (index < 0 || index >= buttons.Length)
            {
                Debug.LogError("Index out of bounds: " + index);
                return;
            }
            if (buttonText != null)
            {
                Debug.Log("Button text before change: " + buttonText.text);
            }
            else
            {
                Debug.LogError("Text component not found on button at index: " + index);
                return;
            }

            if (buttonText.text == "")
            {
                //buttonText.text = "Test";
                buttonText.text = currentPlayer; // 更新按钮文本



                Debug.Log("Current button text: " + buttonText.text); // 检查当前文本

                moveCount++; // 增加移动计数
                CheckWinCondition(); // 检查胜利条件

                // 如果是单人游戏且当前玩家是 "X"
                if (isSinglePlayer && currentPlayer == "X")
                {
                    currentPlayer = "O"; // 切换到 AI
                    AITurn(); // 进行 AI 行动
                }
                else
                {
                    // 切换玩家
                    currentPlayer = (currentPlayer == "X") ? "O" : "X";
                }
            }
        }

    }

    private void AITurn()
    {
        if (currentPlayer == "O") // 假设AI是"O"
        {
            int index = BestMove();
            OnButtonClick(index);
        }
    }

    private int BestMove()
    {
        int bestScore = int.MinValue;
        int move = -1;
        for (int i = 0; i < buttons.Length; i++)
        {
            if (buttons[i].GetComponentInChildren<Text>().text == "")
            {
                buttons[i].GetComponentInChildren<Text>().text = "O"; // AI的回合
                int score = Minimax(GetBoardState(), 0, false);
                buttons[i].GetComponentInChildren<Text>().text = ""; // 撤销该步
                if (score > bestScore)
                {
                    bestScore = score;
                    move = i;
                }
            }
        }
        return move;
    }

    private int Minimax(string[] board, int depth, bool isMaximizing)
    {
        string winner = CheckWinner(board);
        if (winner == "O") return 10 - depth; // AI获胜
        if (winner == "X") return depth - 10; // 玩家获胜
        if (IsDraw(board)) return 0; // 平局

        if (isMaximizing)
        {
            int bestScore = int.MinValue;
            for (int i = 0; i < board.Length; i++)
            {
                if (board[i] == "")
                {
                    board[i] = "O"; // AI的回合
                    int score = Minimax(board, depth + 1, false);
                    board[i] = ""; // 撤销该步
                    bestScore = Mathf.Max(score, bestScore);
                }
            }
            return bestScore;
        }
        else
        {
            int bestScore = int.MaxValue;
            for (int i = 0; i < board.Length; i++)
            {
                if (board[i] == "")
                {
                    board[i] = "X"; // 玩家回合
                    int score = Minimax(board, depth + 1, true);
                    board[i] = ""; // 撤销该步
                    bestScore = Mathf.Min(score, bestScore);
                }
            }
            return bestScore;
        }
    }

    private string[] GetBoardState()
    {
        string[] board = new string[buttons.Length];
        for (int i = 0; i < buttons.Length; i++)
        {
            board[i] = buttons[i].GetComponentInChildren<Text>().text;
        }
        return board;
    }

    private void CheckWinCondition()
    {
        // 定义胜利条件
        int[,] winConditions = new int[,]
        {
            { 0, 1, 2 }, { 3, 4, 5 }, { 6, 7, 8 }, // 横
            { 0, 3, 6 }, { 1, 4, 7 }, { 2, 5, 8 }, // 竖
            { 0, 4, 8 }, { 2, 4, 6 } // 斜
        };

        for (int i = 0; i < winConditions.GetLength(0); i++)
        {
            if (buttons[winConditions[i, 0]].GetComponentInChildren<Text>().text != "" &&
                buttons[winConditions[i, 0]].GetComponentInChildren<Text>().text ==
                buttons[winConditions[i, 1]].GetComponentInChildren<Text>().text &&
                buttons[winConditions[i, 1]].GetComponentInChildren<Text>().text ==
                buttons[winConditions[i, 2]].GetComponentInChildren<Text>().text)
            {
                // 显示胜利信息

                for (int m = 0; m < 8; m++)
                {
                    buttons[m].interactable = false;

                    winnerAnouncement.text = currentPlayer + "\n" + "  wins!";
                    winnerAnouncement.color = new Color(winnerAnouncement.color.r, winnerAnouncement.color.g, winnerAnouncement.color.b, 255);

                    retry.gameObject.SetActive(true);
                }


                Debug.Log(currentPlayer + " wins!");
                return;
            }
        }

        if (moveCount >= 9)
        {
            Debug.Log("It's a draw!");
            winnerAnouncement.text = "Peace!";
            winnerAnouncement.color = new Color(winnerAnouncement.color.r, winnerAnouncement.color.g, winnerAnouncement.color.b, 255);
            retry.gameObject.SetActive(true);
        }
    }

    private bool IsDraw(string[] board)
    {
        foreach (var cell in board)
        {
            if (cell == "") return false;
        }
        return true;
    }

    private string CheckWinner(string[] board)
    {
        // 判断胜利者
        int[,] winConditions = new int[,]
        {
            { 0, 1, 2 }, { 3, 4, 5 }, { 6, 7, 8 }, // 横
            { 0, 3, 6 }, { 1, 4, 7 }, { 2, 5, 8 }, // 竖
            { 0, 4, 8 }, { 2, 4, 6 } // 斜
        };

        for (int i = 0; i < winConditions.GetLength(0); i++)
        {
            if (board[winConditions[i, 0]] != "" &&
                board[winConditions[i, 0]] == board[winConditions[i, 1]] &&
                board[winConditions[i, 1]] == board[winConditions[i, 2]])
            {
                return board[winConditions[i, 0]];
            }
        }
        return "";
    }


    public void OnSinglePlayerMode()
    {
        isSinglePlayer = true;
        StartGame();
    }

    public void OnMultiplayersMode()
    {
        isSinglePlayer = false;
        StartGame();
    }

    [ContextMenu("Start")]
    public void StartGame()
    {
        for (int i = 0; i < startButtons.Length; i++)
        {
            Debug.Log("HIdding");
            startButtons[i].gameObject.SetActive(false);
        }
        StartCoroutine(FadeOutSlogan(sloganTime));

        StartCoroutine(Showgrid(sloganTime));
        StartCoroutine(FadeInPG(BGTime, sloganTime));


    }
    private IEnumerator FadeInPG(float PGTime, float delay)
    {
        yield return new WaitForSeconds(delay);
        float fadeSpeed = 1f / PGTime; // 根据时间设置变化速度
        float t = PGTime;

        while (t > 0)
        {

            if (playGround.color.a < 1)
            {
                float newAlpha = playGround.color.a + (Time.deltaTime * fadeSpeed);
                playGround.color = new Color(playGround.color.r, playGround.color.g, playGround.color.b, Mathf.Max(newAlpha, 0));
            }

            t -= Time.deltaTime; // 减少时间
            yield return null; // 等待下一帧
        }
    }

    private IEnumerator FadeOutSlogan(float sloganTime)
    {
        float fadeSpeed = 1f / sloganTime; // 根据时间设置变化速度
        float t = sloganTime;

        while (t > 0)
        {
            Debug.Log("start");

            if (slogan.color.a > 0)
            {
                float newAlpha = slogan.color.a - (Time.deltaTime * fadeSpeed);
                slogan.color = new Color(slogan.color.r, slogan.color.g, slogan.color.b, Mathf.Max(newAlpha, 0));
            }

            t -= Time.deltaTime; // 减少时间
            yield return null; // 等待下一帧
        }
    }

    private IEnumerator Showgrid(float delay)
    {
        yield return new WaitForSeconds(delay);
        grid.SetActive(true);
        yield return null;
    }

    public void ResetScene()
    {
        SceneManager.LoadScene(0);
    }

    public void ExitGame()
    {
        Application.Quit();
    }


}
