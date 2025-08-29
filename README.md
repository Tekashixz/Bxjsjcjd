local P,RS,UIS,TS,LP,C,Vim=game:GetService("Players"),game:GetService("RunService"),game:GetService("UserInputService"),game:GetService("TweenService"),game.Players.LocalPlayer,workspace.CurrentCamera,game:GetService("VirtualInputManager")
local isRec=false;local frames={};local recStart=0;local saved={}

local function fmtTime(s)local m=math.floor(s/60) local sec=math.floor(s%60)return string.format("%02d:%02d",m,sec)end
local function notif(txt,col)local n=Instance.new("TextLabel") n.Size=UDim2.new(0,220,0,40) n.Position=UDim2.new(0.5,-110,0.1,0) n.BackgroundColor3=col or Color3.fromRGB(70,70,70) n.TextColor3=Color3.new(1,1,1) n.Text=txt n.TextSize=18 n.Font=Enum.Font.GothamBold n.BackgroundTransparency=0.1 n.Parent=LP:WaitForChild("PlayerGui") local c=Instance.new("UICorner",n) c.CornerRadius=UDim.new(0,12) TS:Create(n,TweenInfo.new(0.5),{Position=UDim2.new(0.5,-110,0.05,0)}):Play() task.delay(1.5,function() TS:Create(n,TweenInfo.new(0.5),{Position=UDim2.new(0.5,-110,0.01,0);Transparency=1}):Play() task.wait(0.5)n:Destroy() end)end

local function startRec() isRec=true;frames={};recStart=tick();notif("🎬 Gravação iniciada",Color3.fromRGB(70,130,180)) end
local function stopRec() isRec=false;notif("⏹ Gravação parada",Color3.fromRGB(220,50,50)) end
local function playRec(f,loop) if not f or #f==0 then return end task.spawn(function() repeat local st=tick() for i,fr in ipairs(f) do local w=(i>1 and fr.Time-f[i-1].Time) or 0 local delta=w-(tick()-st) if delta>0 then task.wait(delta) end if fr.Type=="Camera" then C.CFrame=fr.CFrame elseif fr.Type=="Click" then if fr.GuiObject then local p=fr.GuiObject.AbsolutePosition+fr.GuiObject.AbsoluteSize/2 Vim:SendMouseButtonEvent(p.X,p.Y,0,true,game,1) task.wait(0.05) Vim:SendMouseButtonEvent(p.X,p.Y,0,false,game,1) else Vim:SendMouseButtonEvent(fr.Position.X,fr.Position.Y,0,true,game,1) task.wait(0.05) Vim:SendMouseButtonEvent(fr.Position.X,fr.Position.Y,0,false,game,1) end end end until not loop end) end

RS.RenderStepped:Connect(function() if isRec then table.insert(frames,{Type="Camera",CFrame=C.CFrame,Time=tick()}) end end)
UIS.InputBegan:Connect(function(input,gpe) if isRec and not gpe then if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then local gui=nil local mp=input.Position for _,g in pairs(LP.PlayerGui:GetDescendants()) do if g:IsA("GuiObject") and g.AbsolutePosition and g.AbsoluteSize then local pos=g.AbsolutePosition local sz=g.AbsoluteSize if mp.X>=pos.X and mp.X<=pos.X+sz.X and mp.Y>=pos.Y and mp.Y<=pos.Y+sz.Y then gui=g break end end end table.insert(frames,{Type="Click",Position=input.Position,GuiObject=gui,Time=tick()}) end end end)

-- GUI
local SG=Instance.new("ScreenGui",LP:WaitForChild("PlayerGui"))
local F=Instance.new("Frame",SG)
F.Size=UDim2.new(0,500,0,300);F.Position=UDim2.new(0.5,-250,0.5,-150);F.BackgroundColor3=Color3.fromRGB(28,28,28);F.BorderSizePixel=0
local UC=Instance.new("UICorner",F);UC.CornerRadius=UDim.new(0,18)
local US=Instance.new("UIStroke",F);US.Thickness=2;US.Color=Color3.fromRGB(90,90,90)
local Timer=Instance.new("TextLabel",F);Timer.Size=UDim2.new(0.2,0,0,36);Timer.Position=UDim2.new(0.05,0,0.05,0);Timer.BackgroundTransparency=1;Timer.Text="00:00";Timer.TextColor3=Color3.fromRGB(255,255,255);Timer.TextSize=20;Timer.Font=Enum.Font.GothamBold;Timer.TextXAlignment=Enum.TextXAlignment.Left

spawn(function() while true do if isRec then Timer.Text=fmtTime(tick()-recStart) end task.wait(0.2) end end)

local function btn(text,x,y,cb) local b=Instance.new("TextButton",F) b.Size=UDim2.new(0.18,0,0,36);b.Position=UDim2.new(0,x,0,y);b.BackgroundColor3=Color3.fromRGB(60,60,60);b.Text=text;b.TextColor3=Color3.fromRGB(255,255,255);b.Font=Enum.Font.GothamBold;b.TextSize=18;b.AutoButtonColor=true;local c=Instance.new("UICorner",b);c.CornerRadius=UDim.new(0,12);b.MouseButton1Click:Connect(cb) end

btn("▶ Gravar",20,60,startRec)
btn("■ Parar",120,60,stopRec)
btn("▶ Reproduzir",220,60,function() playRec(frames,false) end)
btn("⟳ Loop Inf.",320,60,function() playRec(frames,true) end)

local SaveBox=Instance.new("TextBox",F);SaveBox.Size=UDim2.new(0.45,0,0,36);SaveBox.Position=UDim2.new(0.05,0,0,200);SaveBox.PlaceholderText="Nome da gravação...";SaveBox.TextColor3=Color3.fromRGB(255,255,255);SaveBox.BackgroundColor3=Color3.fromRGB(50,50,50);Instance.new("UICorner",SaveBox).CornerRadius=UDim.new(0,12)

local Scroll=Instance.new("ScrollingFrame",F);Scroll.Size=UDim2.new(0.9,0,0,80);Scroll.Position=UDim2.new(0.05,0,0,240);Scroll.BackgroundColor3=Color3.fromRGB(35,35,35);Scroll.ScrollBarThickness=8;local UIList=Instance.new("UIListLayout",Scroll);UIList.SortOrder=Enum.SortOrder.LayoutOrder;UIList.Padding=UDim.new(0,6)

local function refresh()
    Scroll:ClearAllChildren()
    UIList.Parent=Scroll
    for n,fm in pairs(saved) do
        local it=Instance.new("Frame",Scroll);it.Size=UDim2.new(1,0,0,36);it.BackgroundTransparency=1
        local lbl=Instance.new("TextLabel",it);lbl.Size=UDim2.new(0.5,0,1,0);lbl.Text=n;lbl.TextColor3=Color3.fromRGB(255,255,255);lbl.BackgroundTransparency=1;lbl.Font=Enum.Font.GothamBold;lbl.TextSize=16
        local p=Instance.new("TextButton",it);p.Size=UDim2.new(0.15,0,1,0);p.Position=UDim2.new(0.5,0,0,0);p.Text="▶";p.BackgroundColor3=Color3.fromRGB(70,130,70);Instance.new("UICorner",p).CornerRadius=UDim.new(0,8);p.MouseButton1Click:Connect(function() playRec(fm,false) end)
        local l=Instance.new("TextButton",it);l.Size=UDim2.new(0.15,0,1,0);l.Position=UDim2.new(0.65,0,0,0);l.Text="⟳";l.BackgroundColor3=Color3.fromRGB(180,90,90);Instance.new("UICorner",l).CornerRadius=UDim.new(0,8);l.MouseButton1Click:Connect(function() playRec(fm,true) end)
        local d=Instance.new("TextButton",it);d.Size=UDim2.new(0.15,0,1,0);d.Position=UDim2.new(0.8,0,0,0);d.Text="❌";d.BackgroundColor3=Color3.fromRGB(150,50,50);Instance.new("UICorner",d).CornerRadius=UDim.new(0,8);d.MouseButton1Click:Connect(function() saved[n]=nil;refresh() end)
    end
end

btn("💾 Salvar",380,200,function()
    local n=SaveBox.Text
    if n~="" and #frames>0 then
        saved[n]=table.clone(frames)
        SaveBox.Text=""
        refresh()
        notif("💾 Gravação salva: "..n)
    end
end)
