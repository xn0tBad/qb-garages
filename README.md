# qb-garages

- Original script: https://github.com/JonasDev99/qb-garages

- This is a modified version of qb-garages made by JonasDev99. 

- Preview: https://streamable.com/4v93tp

- I improved the vehicle tracking system make a system to save personal vehicle location because I hate when cars despawn for some random reason. With this, you can locate your vehicle and when you is close to it, you can spawn it where it was left. It is saving engine / body damage and doors/tyres broken.

## Important
- This is working with renewed phone and brazzers-fakeplates and at the moment is no configurable other way.

## To-Do
- Make it more configurable
- Don't allow it to be spawned if impound or in depot
- Make sure it can't be spawned twice (dont know if it happens)
- Better way of trigger saving coords??
- Make it standalone script (to no depend of qb-garages / renewed phone) || (its possible to adapit easily anyways)

## Installation

Drag 'n Drop replace for qb-garages.

- Delete qb-garages.
- Drag the downloaded qb-garages folder into the [qb] folder.
- Apply patch1.sql to your DB

### If you want my qb-phone edited just Drag 'n Drop replace for qb-phone: https://github.com/xn0tBad/qb-phone


### If you want to modify yours:


1. In qb-phone/client/garage.lua replace RegisterNUICallback('gps-vehicle-garage', function(data, cb) with: 
``` 
RegisterNUICallback('gps-vehicle-garage', function(data, cb)
    local veh = data.veh
    local type = data.type
    TriggerEvent("nb-garages:client:trackCar", veh, type)
end)
```

2. In qb-phone/client/garage.lua ADD: 
``` 
RegisterNetEvent('nb-phone:client:trackSpawnNotification', function(title, text)
    SendNUIMessage({
        action = "PhoneNotification",
        PhoneNotify = {
            title = title,
            text = text,
            icon = "fas fa-car",
            color = "#1DA1F2",
            timeout = 2500,
        },
    })
end)
```

3. In qb-phone/server/garage.lua replace: 
```
QBCore.Functions.CreateCallback('qb-phone:server:GetGarageVehicles', function(source, cb)
    local Player = QBCore.Functions.GetPlayer(source)
    local Vehicles = {}
    local vehdata
    local result = exports.oxmysql:executeSync('SELECT * FROM player_vehicles WHERE citizenid = ?', {Player.PlayerData.citizenid})
    if result[1] then
        for _, v in pairs(result) do
            local VehicleData = QBCore.Shared.Vehicles[v.vehicle]
            local VehicleGarage = "None"
            local enginePercent = round(v.engine / 10, 0)
            local bodyPercent = round(v.body / 10, 0)
            if v.garage then
                if Config.Garages[v.garage] then
                    VehicleGarage = Config.Garages[v.garage]["label"]
                else
                    VehicleGarage = v.garage
                end
            end

            local VehicleState = "In"
            if v.state == 0 then
                VehicleState = "Out"
            elseif v.state == 2 then
                VehicleState = "Impounded"
            end

            if VehicleData["brand"] then
                vehdata = {
                    fullname = VehicleData["brand"] .. " " .. VehicleData["name"],
                    brand = VehicleData["brand"],
                    model = VehicleData["name"],
                    plate = v.plate,
                    garage = VehicleGarage,
                    state = VehicleState,
                    fuel = v.fuel,
                    engine = enginePercent,
                    body = bodyPercent,
                    paymentsleft = v.paymentsleft,
                    x = v.x,
                    y = v.y,
                    z = v.z,
                    h = v.h,
                    vehicle = v.vehicle,
                    damage = v.damage,
                    fakeplate = v.fakeplate or nil
                }
            else
                vehdata = {
                    fullname = VehicleData["name"],
                    brand = VehicleData["name"],
                    model = VehicleData["name"],
                    plate = v.plate,
                    garage = VehicleGarage,
                    state = VehicleState,
                    fuel = v.fuel,
                    engine = enginePercent,
                    body = bodyPercent,
                    paymentsleft = v.paymentsleft,
                    x = v.x,
                    y = v.y,
                    z = v.z,
                    h = v.h,
                    vehicle = v.vehicle,
                    damage = v.damage,
                    fakeplate = v.fakeplate or nil
                }
            end
            Vehicles[#Vehicles+1] = vehdata
        end
        cb(Vehicles)
    else
        cb(nil)
    end
end)
```

4. In qb-phone/html/css/garage.lua add and replace:
```
.box-track{
    background:#e0e278;
    padding: 4.0%;
}

.box-spawn{
    background:#90e278;
    padding: 4.0%;
}
```

5. In qb-phone/html/js/garage.js add and replace:
```
$(document).on('click', '.box-track', function(e){
    e.preventDefault()
    $.post("https://qb-phone/gps-vehicle-garage", JSON.stringify({
        veh: veh,
        type: 0,
    }));
});

$(document).on('click', '.box-spawn', function(e){
    e.preventDefault()
    $.post("https://qb-phone/gps-vehicle-garage", JSON.stringify({
        veh: veh,
        type: 1,
    }));
});
```

6. In qb-phone/html/js/garage.js replace:
```
SetupGarageVehicles = function(Vehicles) {
    $(".garage-vehicles").html("");
    if (Vehicles != null) {
        $.each(Vehicles, function(i, vehicle){
            var Element = '<div class="garage-vehicle" id="vehicle-'+i+'"><span class="garage-vehicle-icon"><i class="fas fa-car"></i></span> <span class="garage-vehicle-name">'+vehicle.fullname+'</span> <span class="garage-plate-name">'+vehicle.plate+'</span> <span class="garage-state-name">'+vehicle.state+'</span>' +
            '<div class="garage-block">' +
                '<div class="garage-name"><i class="fas fa-map-marker-alt"></i>'+vehicle.garage+'</div>' +
                '<div class="garage-plate"><i class="fas fa-closed-captioning"></i>'+vehicle.plate+'</div>' +
                '<div class="garage-fuel"><i class="fas fa-gas-pump"></i>'+vehicle.fuel+'</div>' +
                '<div class="garage-engine"><i class="fas fa-oil-can"></i>'+vehicle.engine + " %"+'</div>' +
                '<div class="garage-body"><i class="fas fa-car-crash"></i>'+vehicle.body+ " %"+'</div>' +
                '<div class="garage-payments"><i class="fas fa-hand-holding-usd"></i>'+vehicle.paymentsleft+' Payments Left</div>' +
                '<div class="garage-box" id="'+vehicle.plate+'"><span class="garage-box box-track" style="margin-left: 0.5vh;">TRACK</span><span class="garage-box box-spawn" style="margin-left: 0.5vh;">SPAWN</span><span class="garage-box box-sellvehicle" style = "margin-left: 0.5vh;">SELL</span></div>' +
            '</div>' +
            '</div>';

            $(".garage-vehicles").append(Element);
            $("#vehicle-"+i).data('VehicleData', vehicle);
        });
    }
}
```

## Contact

- Discord: n0t baD#6511
