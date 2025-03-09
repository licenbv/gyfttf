local replicated_storage = game.GetService(game, "ReplicatedStorage");
local players = game.GetService(game, "Players");

local camera = workspace.CurrentCamera;
local utility = require(replicated_storage.Modules.Utility);

local get_players = function() -- this is dumb asf, feel free to modify.
    local entities = {};

    for _, child in workspace.GetChildren(workspace) do
        if child.FindFirstChildOfClass(child, "Humanoid") then
            table.insert(entities, child);
        elseif child.Name == "HurtEffect" then
            for _, hurt_player in child.GetChildren(child) do
                if (hurt_player.ClassName ~= "Highlight") then
                    table.insert(entities, hurt_player);
                end
            end
        end
    end
    return entities
end
local get_closest_player = function()
    local closest, closest_distance = nil, math.huge;
    local character = players.LocalPlayer.Character;

    if (character == nil) then
        return;
    end

    for _, player in get_players() do
        if (player == players.LocalPlayer) then
            continue;
        end

        if (not player:FindFirstChild("HumanoidRootPart")) then
            continue;
        end

        local position, on_screen = camera.WorldToViewportPoint(camera, player.HumanoidRootPart.Position);

        if (on_screen == false) then
            continue;
        end

        local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2);
        local distance = (center - Vector2.new(position.X, position.Y)).Magnitude;

        if (distance > closest_distance) then
            continue;
        end

        closest = player;
        closest_distance = distance;
    end
    return closest;
end

local old = utility.Raycast; utility.Raycast = function(...)
    local arguments = {...};

    if (#arguments > 0 and arguments[4] == 999) then
        local closest = get_closest_player();

        if (closest) then
            arguments[3] = closest.Head.Position;
        end
    end
    return old(table.unpack(arguments));
end
