local p = game:GetService("Players").LocalPlayer
local pg = p:WaitForChild("PlayerGui")
if pg:FindFirstChild("WordleFinal") then pg.WordleFinal:Destroy() end

local sg = Instance.new("ScreenGui", pg); sg.Name = "WordleFinal"

-- ГЛАВНОЕ ОКНО
local main = Instance.new("Frame", sg)
main.Size = UDim2.new(0, 400, 0, 650)
main.Position = UDim2.new(0.5, -200, 0.5, -325)
main.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
main.Active = true; main.Draggable = true
Instance.new("UICorner", main)

local frameStroke = Instance.new("UIStroke", main)
frameStroke.Color = Color3.fromRGB(100, 100, 100)
frameStroke.Thickness = 1

-- Кнопка закрытия
local closeBtn = Instance.new("TextButton", main)
closeBtn.Size = UDim2.new(0, 35, 0, 35); closeBtn.Position = UDim2.new(1, -40, 0, 5)
closeBtn.Text = "X"; closeBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
closeBtn.TextColor3 = Color3.new(1, 1, 1); closeBtn.Font = "GothamBold"
Instance.new("UICorner", closeBtn)
closeBtn.MouseButton1Click:Connect(function() sg:Destroy() end)

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, 0, 0, 60); title.Text = "WORDLE"
title.TextColor3 = Color3.new(1, 1, 1); title.Font = "GothamBold"; title.TextSize = 30; title.BackgroundTransparency = 1

-- Сетка игры
local grid = Instance.new("Frame", main)
grid.Size = UDim2.new(0, 260, 0, 310); grid.Position = UDim2.new(0.5, -130, 0, 70)
grid.BackgroundTransparency = 1
local layout = Instance.new("UIGridLayout", grid)
layout.CellSize = UDim2.new(0, 48, 0, 48); layout.CellPadding = UDim2.new(0, 5, 0, 5)

local cells = {}
for i = 1, 30 do
    local cell = Instance.new("TextLabel", grid)
    cell.Text = ""; cell.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    cell.TextColor3 = Color3.new(1, 1, 1); cell.Font = "GothamBold"; cell.TextSize = 26
    local s = Instance.new("UIStroke", cell)
    s.Color = Color3.fromRGB(80, 80, 80); s.Thickness = 2
    Instance.new("UICorner", cell)
    cells[i] = cell
end

-- Клавиатура
local kb = Instance.new("Frame", main)
kb.Size = UDim2.new(1, -20, 0, 160); kb.Position = UDim2.new(0, 10, 0, 400); kb.BackgroundTransparency = 1
local kLayout = Instance.new("UIGridLayout", kb)
kLayout.CellSize = UDim2.new(0, 33, 0, 40); kLayout.CellPadding = UDim2.new(0, 4, 0, 4)

local dictionary = {"АРБУЗ", "БАНКА", "ВЕТЕР", "ГОРОД", "ДВЕРЬ", "ЗАМОК", "ИГРОК", "КНИГА", "ЛОДКА", "МЕТРО", "НОМЕР", "ОКЕАН", "ПЕСОК", "РЕЧКА", "СПОРТ", "ТУМАН", "ШКОЛА", "ЭКРАН", "ЯГОДА", "ПОЕЗД", "ПЛИТА", "РУЧКА", "СЛОВО", "ФАКЕЛ", "ПОЧТА", "ОБЛАКО", "ЭФИР", "ТЕКСТ", "ПРАВО"}
local alphabet = "АБВГДЕЁЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯ"
local currentWord, currentAttempt, guess = "", 1, ""
local isGameOver = false

function getChars(str)
    local chars = {}
    for c in str:gmatch("[%z\1-\127\194-\244][\128-\191]*") do table.insert(chars, c) end
    return chars
end

function startNewGame()
    currentWord = dictionary[math.random(#dictionary)]
    currentAttempt = 1; guess = ""; isGameOver = false
    title.Text = "WORDLE"; title.TextColor3 = Color3.new(1, 1, 1)
    for _, cell in ipairs(cells) do
        cell.Text = ""; cell.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        cell.UIStroke.Color = Color3.fromRGB(80, 80, 80)
    end
end

function check()
    if isGameOver then return end
    local gChars = getChars(guess)
    if #gChars < 5 then return end
    
    local targetChars = getChars(currentWord)
    local startIdx = (currentAttempt - 1) * 5
    local matched = {}

    for i = 1, 5 do
        local cell = cells[startIdx + i]
        if gChars[i] == targetChars[i] then
            cell.BackgroundColor3 = Color3.fromRGB(83, 141, 78)
            cell.UIStroke.Color = Color3.fromRGB(83, 141, 78)
            matched[i] = true
        end
    end
    
    for i = 1, 5 do
        local cell = cells[startIdx + i]
        if gChars[i] ~= targetChars[i] then
            local found = false
            for j = 1, 5 do
                if not matched[j] and gChars[i] == targetChars[j] then
                    cell.BackgroundColor3 = Color3.fromRGB(181, 159, 59)
                    cell.UIStroke.Color = Color3.fromRGB(181, 159, 59)
                    matched[j] = true; found = true; break
                end
            end
            if not found then
                cell.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
                cell.UIStroke.Color = Color3.fromRGB(70, 70, 70)
            end
        end
    end

    if guess == currentWord then
        title.Text = "ПОБЕДА!"; title.TextColor3 = Color3.new(0, 1, 0)
        isGameOver = true
    elseif currentAttempt >= 6 then
        title.Text = "ОТВЕТ: " .. currentWord; title.TextColor3 = Color3.new(1, 0, 0)
        isGameOver = true
    else
        currentAttempt = currentAttempt + 1; guess = ""
    end
end

-- Клавиши букв
for _, char in ipairs(getChars(alphabet)) do
    local btn = Instance.new("TextButton", kb)
    btn.Text = char; btn.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    btn.TextColor3 = Color3.new(1, 1, 1); btn.Font = "GothamBold"; btn.TextSize = 14
    Instance.new("UICorner", btn)
    btn.MouseButton1Click:Connect(function()
        if isGameOver then return end
        if #getChars(guess) < 5 then
            guess = guess .. char
            cells[(currentAttempt - 1) * 5 + #getChars(guess)].Text = char
        end
    end)
end

-- КНОПКИ УПРАВЛЕНИЯ
local btnFrame = Instance.new("Frame", main)
btnFrame.Size = UDim2.new(1, 0, 0, 50); btnFrame.Position = UDim2.new(0, 0, 0, 565); btnFrame.BackgroundTransparency = 1

local enter = Instance.new("TextButton", btnFrame)
enter.Size = UDim2.new(0, 90, 0, 40); enter.Position = UDim2.new(0.5, -45, 0, 0)
enter.Text = "ВВОД"; enter.BackgroundColor3 = Color3.fromRGB(83, 141, 78); enter.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", enter); enter.MouseButton1Click:Connect(check)

local del = Instance.new("TextButton", btnFrame)
del.Size = UDim2.new(0, 90, 0, 40); del.Position = UDim2.new(0, 15, 0, 0)
del.Text = "СТЕРЕТЬ"; del.BackgroundColor3 = Color3.fromRGB(120, 40, 40); del.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", del)
del.MouseButton1Click:Connect(function()
    if isGameOver then return end
    local gChars = getChars(guess)
    if #gChars > 0 then
        cells[(currentAttempt - 1) * 5 + #gChars].Text = ""
        local newGuess = ""
        for i = 1, #gChars - 1 do newGuess = newGuess .. gChars[i] end
        guess = newGuess
    end
end)

local restart = Instance.new("TextButton", btnFrame)
restart.Size = UDim2.new(0, 110, 0, 40); restart.Position = UDim2.new(1, -125, 0, 0)
restart.Text = "НОВАЯ ИГРА"; restart.BackgroundColor3 = Color3.fromRGB(50, 50, 150); restart.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", restart); restart.MouseButton1Click:Connect(startNewGame)

startNewGame()
