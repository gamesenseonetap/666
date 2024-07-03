# 666
111
local err = false
local requiredLibs = {
    [1] = {Module = "gamesense/images", Link = "https://gamesense.pub/forums/viewtopic.php?id=22917"}
}
for k in ipairs(requiredLibs) do
    if (not pcall(require, requiredLibs[k].Module)) then
        local base = requiredLibs[k]
        if (not err) then
            err = true
        end
        print(string.format("Missing Module: %s, here: %s", base.Module, base.Link))
    end
end
if (err) then
    error("read above [you are missing some module(s)]")
    return
end
if entity.get_prop(entity.get_game_rules(), "m_bIsValveDS") == 1 then
    error(string.format("[genesis] valve server detected!"))
    return
end
local versions = "4.2.0"
local images = require "gamesense/images"
local get_screen_size, get_latency, get_tickinterval = client.screen_size, client.latency, globals.tickinterval
local sw, sh = get_screen_size()
local tst = panorama.open().MyPersonaAPI.GetXuid()
local TabIndexOverflow = function(seed, table)
    for i = 1, #table do
        if seed - table[i] <= 0 then
            return i, seed
        end
        seed = seed - table[i]
    end
end
local GetDate = function()
    local unix = client.unix_time()
    assert(
        unix == nil or type(unix) == "number" or unix:find("/Date%((%d+)"),
        'Please input a valid number to "getDate"'
    )
    unix = (type(unix) == "string" and unix:match("/Date%((%d+)") / 1000 or unix or os.time())
    local dayCount, year, days, month = function(yr)
            return (yr % 4 == 0 and (yr % 100 ~= 0 or yr % 400 == 0)) and 366 or 365
        end, 1970, math.ceil(unix / 86400)
    while days >= dayCount(year) do
        days = days - dayCount(year)
        year = year + 1
    end
    month, days =
        TabIndexOverflow(days, {31, (dayCount(year) == 366 and 29 or 28), 31, 30, 31, 30, 31, 31, 30, 31, 30, 31})
    return days, month, year
end
local checkFormat = function(data)
    return (data < 10) and "0" .. data or data
end
local GetDiscordTimeFormatted = function()
    local day, month, year = GetDate()
    local hours, minutes, seconds = client.system_time()
    return string.format(
        "%s-%s-%s %s:%s:%s",
        checkFormat(year),
        checkFormat(month),
        checkFormat(day),
        checkFormat(hours),
        checkFormat(minutes),
        checkFormat(seconds)
    )
end

get_latency()
get_tickinterval()
local dragging = (function()
    local a = {}
    local b, c, d, e, f, g, h, i, j, k, l, m, n, o
    local p = {__index = {drag = function(self, ...)
                local q, r = self:get()
                local s, t = a.drag(q, r, ...)
                if q ~= s or r ~= t then
                    self:set(s, t)
                end
                return s, t
            end, set = function(self, q, r)
                local j, k = client.screen_size()
                ui.set(self.x_reference, q / j * self.res)
                ui.set(self.y_reference, r / k * self.res)
            end, get = function(self)
                local j, k = client.screen_size()
                return ui.get(self.x_reference) / self.res * j, ui.get(self.y_reference) / self.res * k
            end}}
    function a.new(u, v, w, x)
        x = x or 10000
        local j, k = client.screen_size()
        local y = ui.new_slider("LUA", "A", u .. " window position", 0, x, v / j * x)
        local z = ui.new_slider("LUA", "A", "\n" .. u .. " window position y", 0, x, w / k * x)
        ui.set_visible(y, false)
        ui.set_visible(z, false)
        return setmetatable({name = u, x_reference = y, y_reference = z, res = x}, p)
    end
    function a.drag(q, r, A, B, C, D, E)
        if globals.framecount() ~= b then
            c = ui.is_menu_open()
            f, g = d, e
            d, e = ui.mouse_position()
            i = h
            h = client.key_state(1) == true
            m = l
            l = {}
            o = n
            n = false
            j, k = client.screen_size()
        end
        if c and i ~= nil then
            if (not i or o) and h and f > q and g > r and f < q + A and g < r + B then
                n = true
                q, r = q + d - f, r + e - g
                if not D then
                    q = math.max(0, math.min(j - A, q))
                    r = math.max(0, math.min(k - B, r))
                end
            end
        end
        table.insert(l, {q, r, A, B})
        return q, r, A, B
    end
    return a
end)()
function math.round(number, precision)
    local mult = 10 ^ (precision or 0)
    return math.floor(number * mult + 0.5) / mult
end
local angle_c = {}
local angle_mt = {__index = angle_c}
angle_mt.__call = function(angle, p_new, y_new, r_new)
    p_new = p_new or angle.p
    y_new = y_new or angle.y
    r_new = r_new or angle.r
    angle.p = p_new
    angle.y = y_new
    angle.r = r_new
end
local function angle(p, y, r)
    return setmetatable({p = p and p or 0, y = y and y or 0, r = r and r or 0}, angle_mt)
end
function angle_c:set(p, y, r)
    p = p or self.p
    y = y or self.y
    r = r or self.r
    self.p = p
    self.y = y
    self.r = r
end
function angle_c:offset(p, y, r)
    p = self.p + p or 0
    y = self.y + y or 0
    r = self.r + r or 0
    self.p = self.p + p
    self.y = self.y + y
    self.r = self.r + r
end
function angle_c:clone()
    return setmetatable({p = self.p, y = self.y, r = self.r}, angle_mt)
end
function angle_c:clone_offset(p, y, r)
    p = self.p + p or 0
    y = self.y + y or 0
    r = self.r + r or 0
    return angle(self.p + p, self.y + y, self.r + r)
end
function angle_c:clone_set(p, r, r)
    p = p or self.p
    r = r or self.y
    r = r or self.r
    return angle(p, r, r)
end
function angle_c:unpack()
    return self.p, self.y, self.r
end
function angle_c:nullify()
    self.p = 0
    self.y = 0
    self.r = 0
end
function angle_mt.__tostring(operand_a)
    return string.format("%s, %s, %s", operand_a.p, operand_a.y, operand_a.r)
end
function angle_mt.__concat(operand_a)
    return string.format("%s, %s, %s", operand_a.p, operand_a.y, operand_a.r)
end
function angle_mt.__add(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return angle(operand_a + operand_b.p, operand_a + operand_b.y, operand_a + operand_b.r)
    end
    if (type(operand_b) == "number") then
        return angle(operand_a.p + operand_b, operand_a.y + operand_b, operand_a.r + operand_b)
    end
    return angle(operand_a.p + operand_b.p, operand_a.y + operand_b.y, operand_a.r + operand_b.r)
end
function angle_mt.__sub(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return angle(operand_a - operand_b.p, operand_a - operand_b.y, operand_a - operand_b.r)
    end
    if (type(operand_b) == "number") then
        return angle(operand_a.p - operand_b, operand_a.y - operand_b, operand_a.r - operand_b)
    end
    return angle(operand_a.p - operand_b.p, operand_a.y - operand_b.y, operand_a.r - operand_b.r)
end
function angle_mt.__mul(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return angle(operand_a * operand_b.p, operand_a * operand_b.y, operand_a * operand_b.r)
    end
    if (type(operand_b) == "number") then
        return angle(operand_a.p * operand_b, operand_a.y * operand_b, operand_a.r * operand_b)
    end
    return angle(operand_a.p * operand_b.p, operand_a.y * operand_b.y, operand_a.r * operand_b.r)
end
function angle_mt.__div(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return angle(operand_a / operand_b.p, operand_a / operand_b.y, operand_a / operand_b.r)
    end
    if (type(operand_b) == "number") then
        return angle(operand_a.p / operand_b, operand_a.y / operand_b, operand_a.r / operand_b)
    end
    return angle(operand_a.p / operand_b.p, operand_a.y / operand_b.y, operand_a.r / operand_b.r)
end
function angle_mt.__pow(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return angle(
            math.pow(operand_a, operand_b.p),
            math.pow(operand_a, operand_b.y),
            math.pow(operand_a, operand_b.r)
        )
    end
    if (type(operand_b) == "number") then
        return angle(
            math.pow(operand_a.p, operand_b),
            math.pow(operand_a.y, operand_b),
            math.pow(operand_a.r, operand_b)
        )
    end
    return angle(
        math.pow(operand_a.p, operand_b.p),
        math.pow(operand_a.y, operand_b.y),
        math.pow(operand_a.r, operand_b.r)
    )
end
function angle_mt.__mod(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return angle(operand_a % operand_b.p, operand_a % operand_b.y, operand_a % operand_b.r)
    end
    if (type(operand_b) == "number") then
        return angle(operand_a.p % operand_b, operand_a.y % operand_b, operand_a.r % operand_b)
    end
    return angle(operand_a.p % operand_b.p, operand_a.y % operand_b.y, operand_a.r % operand_b.r)
end
function angle_mt.__unm(operand_a)
    return angle(-operand_a.p, -operand_a.y, -operand_a.r)
end
function angle_c:round_zero()
    self.p = math.floor(self.p + 0.5)
    self.y = math.floor(self.y + 0.5)
    self.r = math.floor(self.r + 0.5)
end
function angle_c:round(precision)
    self.p = math.round(self.p, precision)
    self.y = math.round(self.y, precision)
    self.r = math.round(self.r, precision)
end
function angle_c:round_base(base)
    self.p = base * math.round(self.p / base)
    self.y = base * math.round(self.y / base)
    self.r = base * math.round(self.r / base)
end
function angle_c:rounded_zero()
    return angle(math.floor(self.p + 0.5), math.floor(self.y + 0.5), math.floor(self.r + 0.5))
end
function angle_c:rounded(precision)
    return angle(math.round(self.p, precision), math.round(self.y, precision), math.round(self.r, precision))
end
function angle_c:rounded_base(base)
    return angle(base * math.round(self.p / base), base * math.round(self.y / base), base * math.round(self.r / base))
end
local vector_c = {}
local vector_mt = {__index = vector_c}
vector_mt.__call = function(vector, x_new, y_new, z_new)
    x_new = x_new or vector.x
    y_new = y_new or vector.y
    z_new = z_new or vector.z
    vector.x = x_new
    vector.y = y_new
    vector.z = z_new
end
local function vector(x, y, z)
    return setmetatable({x = x and x or 0, y = y and y or 0, z = z and z or 0}, vector_mt)
end
function vector_c:set(x_new, y_new, z_new)
    x_new = x_new or self.x
    y_new = y_new or self.y
    z_new = z_new or self.z
    self.x = x_new
    self.y = y_new
    self.z = z_new
end
function vector_c:offset(x_offset, y_offset, z_offset)
    x_offset = x_offset or 0
    y_offset = y_offset or 0
    z_offset = z_offset or 0
    self.x = self.x + x_offset
    self.y = self.y + y_offset
    self.z = self.z + z_offset
end
function vector_c:clone()
    return setmetatable({x = self.x, y = self.y, z = self.z}, vector_mt)
end
function vector_c:clone_offset(x_offset, y_offset, z_offset)
    x_offset = x_offset or 0
    y_offset = y_offset or 0
    z_offset = z_offset or 0
    return setmetatable({x = self.x + x_offset, y = self.y + y_offset, z = self.z + z_offset}, vector_mt)
end
function vector_c:clone_set(x_new, y_new, z_new)
    x_new = x_new or self.x
    y_new = y_new or self.y
    z_new = z_new or self.z
    return vector(x_new, y_new, z_new)
end
function vector_c:unpack()
    return self.x, self.y, self.z
end
function vector_c:nullify()
    self.x = 0
    self.y = 0
    self.z = 0
end
function vector_mt.__tostring(operand_a)
    return string.format("%s, %s, %s", operand_a.x, operand_a.y, operand_a.z)
end
function vector_mt.__concat(operand_a)
    return string.format("%s, %s, %s", operand_a.x, operand_a.y, operand_a.z)
end
function vector_mt.__eq(operand_a, operand_b)
    return (operand_a.x == operand_b.x) and (operand_a.y == operand_b.y) and (operand_a.z == operand_b.z)
end
function vector_mt.__lt(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return (operand_a < operand_b.x) or (operand_a < operand_b.y) or (operand_a < operand_b.z)
    end
    if (type(operand_b) == "number") then
        return (operand_a.x < operand_b) or (operand_a.y < operand_b) or (operand_a.z < operand_b)
    end
    return (operand_a.x < operand_b.x) or (operand_a.y < operand_b.y) or (operand_a.z < operand_b.z)
end
function vector_mt.__le(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return (operand_a <= operand_b.x) or (operand_a <= operand_b.y) or (operand_a <= operand_b.z)
    end
    if (type(operand_b) == "number") then
        return (operand_a.x <= operand_b) or (operand_a.y <= operand_b) or (operand_a.z <= operand_b)
    end
    return (operand_a.x <= operand_b.x) or (operand_a.y <= operand_b.y) or (operand_a.z <= operand_b.z)
end
function vector_mt.__add(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return vector(operand_a + operand_b.x, operand_a + operand_b.y, operand_a + operand_b.z)
    end
    if (type(operand_b) == "number") then
        return vector(operand_a.x + operand_b, operand_a.y + operand_b, operand_a.z + operand_b)
    end
    return vector(operand_a.x + operand_b.x, operand_a.y + operand_b.y, operand_a.z + operand_b.z)
end
function vector_mt.__sub(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return vector(operand_a - operand_b.x, operand_a - operand_b.y, operand_a - operand_b.z)
    end
    if (type(operand_b) == "number") then
        return vector(operand_a.x - operand_b, operand_a.y - operand_b, operand_a.z - operand_b)
    end
    return vector(operand_a.x - operand_b.x, operand_a.y - operand_b.y, operand_a.z - operand_b.z)
end
function vector_mt.__mul(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return vector(operand_a * operand_b.x, operand_a * operand_b.y, operand_a * operand_b.z)
    end
    if (type(operand_b) == "number") then
        return vector(operand_a.x * operand_b, operand_a.y * operand_b, operand_a.z * operand_b)
    end
    return vector(operand_a.x * operand_b.x, operand_a.y * operand_b.y, operand_a.z * operand_b.z)
end
function vector_mt.__div(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return vector(operand_a / operand_b.x, operand_a / operand_b.y, operand_a / operand_b.z)
    end
    if (type(operand_b) == "number") then
        return vector(operand_a.x / operand_b, operand_a.y / operand_b, operand_a.z / operand_b)
    end
    return vector(operand_a.x / operand_b.x, operand_a.y / operand_b.y, operand_a.z / operand_b.z)
end
function vector_mt.__pow(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return vector(
            math.pow(operand_a, operand_b.x),
            math.pow(operand_a, operand_b.y),
            math.pow(operand_a, operand_b.z)
        )
    end
    if (type(operand_b) == "number") then
        return vector(
            math.pow(operand_a.x, operand_b),
            math.pow(operand_a.y, operand_b),
            math.pow(operand_a.z, operand_b)
        )
    end
    return vector(
        math.pow(operand_a.x, operand_b.x),
        math.pow(operand_a.y, operand_b.y),
        math.pow(operand_a.z, operand_b.z)
    )
end
function vector_mt.__mod(operand_a, operand_b)
    if (type(operand_a) == "number") then
        return vector(operand_a % operand_b.x, operand_a % operand_b.y, operand_a % operand_b.z)
    end
    if (type(operand_b) == "number") then
        return vector(operand_a.x % operand_b, operand_a.y % operand_b, operand_a.z % operand_b)
    end
    return vector(operand_a.x % operand_b.x, operand_a.y % operand_b.y, operand_a.z % operand_b.z)
end
function vector_mt.__unm(operand_a)
    return vector(-operand_a.x, -operand_a.y, -operand_a.z)
end
function vector_c:length2_squared()
    return (self.x * self.x) + (self.y * self.y)
end
function vector_c:length2()
    return math.sqrt(self:length2_squared())
end
function vector_c:length_squared()
    return (self.x * self.x) + (self.y * self.y) + (self.z * self.z)
end
function vector_c:length()
    return math.sqrt(self:length_squared())
end
function vector_c:dot_product(b)
    return (self.x * b.x) + (self.y * b.y) + (self.z * b.z)
end
function vector_c:cross_product(b)
    return vector((self.y * b.z) - (self.z * b.y), (self.z * b.x) - (self.x * b.z), (self.x * b.y) - (self.y * b.x))
end
function vector_c:distance2(destination)
    return (destination - self):length2()
end
function vector_c:distance(destination)
    return (destination - self):length()
end
function vector_c:distance_x(destination)
    return math.abs(self.x - destination.x)
end
function vector_c:distance_y(destination)
    return math.abs(self.y - destination.y)
end
function vector_c:distance_z(destination)
    return math.abs(self.z - destination.z)
end
function vector_c:in_range(destination, distance)
    return self:distance(destination) <= distance
end
function vector_c:round_zero()
    self.x = math.floor(self.x + 0.5)
    self.y = math.floor(self.y + 0.5)
    self.z = math.floor(self.z + 0.5)
end
function vector_c:round(precision)
    self.x = math.round(self.x, precision)
    self.y = math.round(self.y, precision)
    self.z = math.round(self.z, precision)
end
function vector_c:round_base(base)
    self.x = base * math.round(self.x / base)
    self.y = base * math.round(self.y / base)
    self.z = base * math.round(self.z / base)
end
function vector_c:rounded_zero()
    return vector(math.floor(self.x + 0.5), math.floor(self.y + 0.5), math.floor(self.z + 0.5))
end
function vector_c:rounded(precision)
    return vector(math.round(self.x, precision), math.round(self.y, precision), math.round(self.z, precision))
end
function vector_c:rounded_base(base)
    return vector(base * math.round(self.x / base), base * math.round(self.y / base), base * math.round(self.z / base))
end
function vector_c:normalize()
    local length = self:length()
    if (length ~= 0) then
        self.x = self.x / length
        self.y = self.y / length
        self.z = self.z / length
    else
        self.x = 0
        self.y = 0
        self.z = 1
    end
end
function vector_c:normalized_length()
    return self:length()
end
function vector_c:normalized()
    local length = self:length()
    if (length ~= 0) then
        return vector(self.x / length, self.y / length, self.z / length)
    else
        return vector(0, 0, 1)
    end
end
function vector_c:to_screen(only_within_screen_boundary)
    local x, y = renderer.world_to_screen(self.x, self.y, self.z)
    if (x == nil or y == nil) then
        return nil
    end
    if (only_within_screen_boundary == true) then
        local screen_x, screen_y = client.screen_size()
        if (x < 0 or x > screen_x or y < 0 or y > screen_y) then
            return nil
        end
    end
    return vector(x, y)
end
function vector_c:magnitude()
    return math.sqrt(math.pow(self.x, 2) + math.pow(self.y, 2) + math.pow(self.z, 2))
end
function vector_c:angle_to(destination)
    local delta_vector = vector(destination.x - self.x, destination.y - self.y, destination.z - self.z)
    local yaw = math.deg(math.atan2(delta_vector.y, delta_vector.x))
    local hyp = math.sqrt(delta_vector.x * delta_vector.x + delta_vector.y * delta_vector.y)
    local pitch = math.deg(math.atan2(-delta_vector.z, hyp))
    return angle(pitch, yaw)
end
function vector_c:lerp(destination, percentage)
    return self + (destination - self) * percentage
end
local function vector_internal_division(source, destination, m, n)
    return vector(
        (source.x * n + destination.x * m) / (m + n),
        (source.y * n + destination.y * m) / (m + n),
        (source.z * n + destination.z * m) / (m + n)
    )
end
function vector_c:trace_line_to(destination, skip_entindex)
    skip_entindex = skip_entindex or -1
    return client.trace_line(skip_entindex, self.x, self.y, self.z, destination.x, destination.y, destination.z)
end
function vector_c:trace_line_impact(destination, skip_entindex)
    skip_entindex = skip_entindex or -1
    local fraction, eid =
        client.trace_line(skip_entindex, self.x, self.y, self.z, destination.x, destination.y, destination.z)
    local impact = self:lerp(destination, fraction)
    return fraction, eid, impact
end
function vector_c:trace_line_skip_indices(destination, max_traces, callback)
    max_traces = max_traces or 10
    local fraction, eid = 0, -1
    local impact = self
    local i = 0
    while (max_traces >= i and fraction < 1 and ((eid > -1 and callback(eid)) or impact == self)) do
        fraction, eid, impact = impact:trace_line_impact(destination, eid)
        i = i + 1
    end
    return self:distance(impact) / self:distance(destination), eid, impact
end
function vector_c:trace_line_skip_class(destination, skip_entindex, skip_distance)
    local should_skip = function(index, skip_entity)
        local class_name = entity.get_classname(index) or ""
        for i in 1, #skip_entity do
            if class_name == skip_entity[i] then
                return true
            end
        end
        return false
    end
    local angles = self:angle_to(destination)
    local direction = angles:to_forward_vector()
    local last_traced_position = self
    while true do
        local fraction, hit_entity = last_traced_position:trace_line_to(destination)
        if fraction == 1 and hit_entity == -1 then
            return 1, -1
        else
            if should_skip(hit_entity, skip_entindex) then
                last_traced_position = vector_internal_division(self, destination, fraction, 1 - fraction)
                last_traced_position = last_traced_position + direction * skip_distance
            else
                return fraction, hit_entity, self:lerp(destination, fraction)
            end
        end
    end
end
function vector_c:trace_bullet_to(destination, eid)
    return client.trace_bullet(eid, self.x, self.y, self.z, destination.x, destination.y, destination.z)
end
function vector_c:closest_ray_point(ray_start, ray_end)
    local to = self - ray_start
    local direction = ray_end - ray_start
    local length = direction:length()
    direction:normalize()
    local ray_along = to:dot_product(direction)
    if (ray_along < 0) then
        return ray_start
    elseif (ray_along > length) then
        return ray_end
    end
    return ray_start + direction * ray_along
end
function vector_c:ray_divided(ray_end, ratio)
    return (self * ratio + ray_end) / (1 + ratio)
end
function vector_c:ray_segmented(ray_end, segments)
    local points = {}
    for i = 0, segments do
        points[i] = vector_internal_division(self, ray_end, i, segments - i)
    end
    return points
end
function vector_c:ray(ray_end, total_segments)
    total_segments = total_segments or 128
    local segments = {}
    local step = self:distance(ray_end) / total_segments
    local angle = self:angle_to(ray_end)
    local direction = angle:to_forward_vector()
    for i = 1, total_segments do
        table.insert(segments, self + (direction * (step * i)))
    end
    local src_screen_position = vector(0, 0, 0)
    local dst_screen_position = vector(0, 0, 0)
    local src_in_screen = false
    local dst_in_screen = false
    for i = 1, #segments do
        src_screen_position = segments[i]:to_screen()
        if src_screen_position ~= nil then
            src_in_screen = true
            break
        end
    end
    for i = #segments, 1, -1 do
        dst_screen_position = segments[i]:to_screen()
        if dst_screen_position ~= nil then
            dst_in_screen = true
            break
        end
    end
    if src_in_screen and dst_in_screen then
        return src_screen_position, dst_screen_position
    end
    return nil
end
function vector_c:ray_intersects_smoke(ray_end)
    return line_goes_through_smoke(self.x, self.y, self.z, ray_end.x, ray_end.y, ray_end.z, 1)
end
function vector_c:inside_polygon2(polygon)
    local odd_nodes = false
    local polygon_vertices = #polygon
    local j = polygon_vertices
    for i = 1, polygon_vertices do
        if (polygon[i].y < self.y and polygon[j].y >= self.y or polygon[j].y < self.y and polygon[i].y >= self.y) then
            if
                (polygon[i].x + (self.y - polygon[i].y) / (polygon[j].y - polygon[i].y) * (polygon[j].x - polygon[i].x) <
                    self.x)
             then
                odd_nodes = not odd_nodes
            end
        end
        j = i
    end
    return odd_nodes
end
local beta_access, login, login_return, discord_log, playsound = "", false, false, false, false
local allow_to_use, username_, allowyes, debug_um, status =
    false,
    "",
    false,
    false,
    "debug"
function vector_c:draw_circle(radius, r, g, b, a, accuracy, width, outline, start_degrees, percentage)
    local accuracy = accuracy ~= nil and accuracy or 3
    local width = width ~= nil and width or 1
    local outline = outline ~= nil and outline or false
    local start_degrees = start_degrees ~= nil and start_degrees or 0
    local percentage = percentage ~= nil and percentage or 1
    local screen_x_line_old, screen_y_line_old
    for rot = start_degrees, percentage * 360, accuracy do
        local rot_temp = math.rad(rot)
        local lineX, lineY, lineZ = radius * math.cos(rot_temp) + self.x, radius * math.sin(rot_temp) + self.y, self.z
        local screen_x_line, screen_y_line = renderer.world_to_screen(lineX, lineY, lineZ)
        if screen_x_line ~= nil and screen_x_line_old ~= nil then
            for i = 1, width do
                local i = i - 1
                renderer.line(screen_x_line, screen_y_line - i, screen_x_line_old, screen_y_line_old - i, r, g, b, a)
            end
            if outline then
                local outline_a = a / 255 * 160
                renderer.line(
                    screen_x_line,
                    screen_y_line - width,
                    screen_x_line_old,
                    screen_y_line_old - width,
                    16,
                    16,
                    16,
                    outline_a
                )
                renderer.line(
                    screen_x_line,
                    screen_y_line + 1,
                    screen_x_line_old,
                    screen_y_line_old + 1,
                    16,
                    16,
                    16,
                    outline_a
                )
            end
        end
        screen_x_line_old, screen_y_line_old = screen_x_line, screen_y_line
    end
end
function vector_c:min(value)
    self.x = math.min(value, self.x)
    self.y = math.min(value, self.y)
    self.z = math.min(value, self.z)
end
function vector_c:max(value)
    self.x = math.max(value, self.x)
    self.y = math.max(value, self.y)
    self.z = math.max(value, self.z)
end
function vector_c:minned(value)
    return vector(math.min(value, self.x), math.min(value, self.y), math.min(value, self.z))
end
function vector_c:maxed(value)
    return vector(math.max(value, self.x), math.max(value, self.y), math.max(value, self.z))
end
function angle_c:to_forward_vector()
    local degrees_to_radians = function(degrees)
        return degrees * math.pi / 180
    end
    local sp = math.sin(degrees_to_radians(self.p))
    local cp = math.cos(degrees_to_radians(self.p))
    local sy = math.sin(degrees_to_radians(self.y))
    local cy = math.cos(degrees_to_radians(self.y))
    return vector(cp * cy, cp * sy, -sp)
end
function angle_c:to_up_vector()
    local degrees_to_radians = function(degrees)
        return degrees * math.pi / 180
    end
    local sp = math.sin(degrees_to_radians(self.p))
    local cp = math.cos(degrees_to_radians(self.p))
    local sy = math.sin(degrees_to_radians(self.y))
    local cy = math.cos(degrees_to_radians(self.y))
    local sr = math.sin(degrees_to_radians(self.r))
    local cr = math.cos(degrees_to_radians(self.r))
    return vector(cr * sp * cy + sr * sy, cr * sp * sy + sr * cy * -1, cr * cp)
end
function angle_c:to_right_vector()
    local degrees_to_radians = function(degrees)
        return degrees * math.pi / 180
    end
    local sp = math.sin(degrees_to_radians(self.p))
    local cp = math.cos(degrees_to_radians(self.p))
    local sy = math.sin(degrees_to_radians(self.y))
    local cy = math.cos(degrees_to_radians(self.y))
    local sr = math.sin(degrees_to_radians(self.r))
    local cr = math.cos(degrees_to_radians(self.r))
    return vector(sr * sp * cy * -1 + cr * sy, sr * sp * sy * -1 + -1 * cr * cy, -1 * sr * cp)
end
function angle_c:to_backward_vector()
    local degrees_to_radians = function(degrees)
        return degrees * math.pi / 180
    end
    local sp = math.sin(degrees_to_radians(self.p))
    local cp = math.cos(degrees_to_radians(self.p))
    local sy = math.sin(degrees_to_radians(self.y))
    local cy = math.cos(degrees_to_radians(self.y))
    return -vector(cp * cy, cp * sy, -sp)
end
function angle_c:to_left_vector()
    local degrees_to_radians = function(degrees)
        return degrees * math.pi / 180
    end
    local sp = math.sin(degrees_to_radians(self.p))
    local cp = math.cos(degrees_to_radians(self.p))
    local sy = math.sin(degrees_to_radians(self.y))
    local cy = math.cos(degrees_to_radians(self.y))
    local sr = math.sin(degrees_to_radians(self.r))
    local cr = math.cos(degrees_to_radians(self.r))
    return -vector(sr * sp * cy * -1 + cr * sy, sr * sp * sy * -1 + -1 * cr * cy, -1 * sr * cp)
end
function angle_c:to_down_vector()
    local degrees_to_radians = function(degrees)
        return degrees * math.pi / 180
    end
    local sp = math.sin(degrees_to_radians(self.p))
    local cp = math.cos(degrees_to_radians(self.p))
    local sy = math.sin(degrees_to_radians(self.y))
    local cy = math.cos(degrees_to_radians(self.y))
    local sr = math.sin(degrees_to_radians(self.r))
    local cr = math.cos(degrees_to_radians(self.r))
    return -vector(cr * sp * cy + sr * sy, cr * sp * sy + sr * cy * -1, cr * cp)
end
function angle_c:fov_to(source, destination)
    local fwd = self:to_forward_vector()
    local delta = (destination - source):normalized()
    local fov = math.acos(fwd:dot_product(delta) / delta:length())
    return math.max(0.0, math.deg(fov))
end
function angle_c:bearing(precision)
    local yaw = 180 - self.y + 90
    local degrees = (yaw % 360 + 360) % 360
    degrees = degrees > 180 and degrees - 360 or degrees
    return math.round(degrees + 180, precision)
end
function angle_c:start_degrees()
    local yaw = self.y
    local degrees = (yaw % 360 + 360) % 360
    degrees = degrees > 180 and degrees - 360 or degrees
    return degrees + 180
end
function angle_c:normalize()
    local pitch = self.p
    if (pitch < -89) then
        pitch = -89
    elseif (pitch > 89) then
        pitch = 89
    end
    local yaw = self.y
    while yaw > 180 do
        yaw = yaw - 360
    end
    while yaw < -180 do
        yaw = yaw + 360
    end
    return angle(pitch, yaw, 0)
end
function angle_c:normalized()
    if (self.p < -89) then
        self.p = -89
    elseif (self.p > 89) then
        self.p = 89
    end
    local yaw = self.y
    while yaw > 180 do
        yaw = yaw - 360
    end
    while yaw < -180 do
        yaw = yaw + 360
    end
    self.y = yaw
    self.r = 0
end
function vector_c.draw_polygon(polygon, r, g, b, a, segments)
    for id, vertex in pairs(polygon) do
        local next_vertex = polygon[id + 1]
        if (next_vertex == nil) then
            next_vertex = polygon[1]
        end
        local ray_a, ray_b = vertex:ray(next_vertex, (segments or 64))
        if (ray_a ~= nil and ray_b ~= nil) then
            renderer.line(ray_a.x, ray_a.y, ray_b.x, ray_b.y, r, g, b, a)
        end
    end
end
function vector_c.eye_position(eid)
    local origin = vector(entity.get_prop(eid, "m_vecOrigin", 3))
    local _, _, view_z = entity.get_prop(eid, "m_vecViewOffset")
    local duck_amount = entity.get_prop(eid, "m_flDuckAmount")
    origin.z = origin.z + view_z - duck_amount * 16
    return origin
end
local M = {}
local table_insert, table_concat, string_rep, string_len, string_sub =
    table.insert,
    table.concat,
    string.rep,
    string.len,
    string.sub
local math_max, math_floor, math_ceil = math.max, math.floor, math.ceil
local function len(str)
    local _, count = string.gsub(tostring(str), "[^\128-\193]", "")
    return count
end
local styles = {
    ["ASCII"] = {"-", "|", "+"},
    ["Compact"] = {"-", " ", " ", " ", " ", " ", " ", " "},
    ["ASCII (Girder)"] = {"=", "||", "//", "[]", "\\\\", "|]", "[]", "[|", "\\\\", "[]", "//"},
    ["Unicode"] = {"═", "║", "╔", "╦", "╗", "╠", "╬", "╣", "╚", "╩", "╝"},
    ["Unicode (Single Line)"] = {"─", "│", "┌", "┬", "┐", "├", "┼", "┤", "└", "┴", "┘"},
    ["Markdown (Github)"] = {"-", "|", "|"}
}
for _, style in pairs(styles) do
    if #style == 3 then
        for j = 4, 11 do
            style[j] = style[3]
        end
    end
end
local function justify_center(text, width)
    text = string_sub(text, 1, width)
    local length = len(text)
    return string_rep(" ", math_floor(width / 2 - length / 2)) ..
        text .. string_rep(" ", math_ceil(width / 2 - length / 2))
end
local function justify_left(text, width)
    text = string_sub(text, 1, width)
    return text .. string_rep(" ", width - len(text))
end
function M.generate_table(rows, headings, options)
    if type(options) == "string" or options == nil then
        options = {style = options or "ASCII"}
    end
    if options.top_line == nil then
        options.top_line = options.style ~= "Markdown (Github)"
    end
    if options.bottom_line == nil then
        options.bottom_line = options.style ~= "Markdown (Github)"
    end
    if options.header_seperator_line == nil then
        options.header_seperator_line = true
    end
    local seperators = styles[options.style] or styles["ASCII"]
    local rows_out, columns_width, columns_count = {}, {}, 0
    local has_headings = headings ~= nil and #headings > 0
    if has_headings then
        for i = 1, #headings do
            columns_width[i] = len(headings[i]) + 2
        end
        columns_count = #headings
    else
        for i = 1, #rows do
            columns_count = math_max(columns_count, #rows[i])
        end
    end
    for i = 1, #rows do
        local row = rows[i]
        for c = 1, columns_count do
            columns_width[c] = math_max(columns_width[c] or 2, len(row[c]) + 2)
        end
    end
    local column_seperator_rows = {}
    for i = 1, columns_count do
        table_insert(column_seperator_rows, string_rep(seperators[1], columns_width[i]))
    end
    if options.top_line then
        table_insert(rows_out, seperators[3] .. table_concat(column_seperator_rows, seperators[4]) .. seperators[5])
    end
    if has_headings then
        local headings_justified = {}
        for i = 1, columns_count do
            headings_justified[i] = justify_center(headings[i], columns_width[i])
        end
        table_insert(rows_out, seperators[2] .. table_concat(headings_justified, seperators[2]) .. seperators[2])
        if options.header_seperator_line then
            table_insert(rows_out, seperators[6] .. table_concat(column_seperator_rows, seperators[7]) .. seperators[8])
        end
    end
    for i = 1, #rows do
        local row, row_out = rows[i], {}
        if #row == 0 then
            table_insert(rows_out, seperators[6] .. table_concat(column_seperator_rows, seperators[7]) .. seperators[8])
        else
            for j = 1, columns_count do
                local justified =
                    options.value_justify == "center" and justify_center(row[j] or "", columns_width[j] - 2) or
                    justify_left(row[j] or "", columns_width[j] - 2)
                row_out[j] = " " .. justified .. " "
            end
            table_insert(rows_out, seperators[2] .. table_concat(row_out, seperators[2]) .. seperators[2])
        end
    end
    if options.bottom_line and seperators[9] then
        table_insert(rows_out, seperators[9] .. table_concat(column_seperator_rows, seperators[10]) .. seperators[11])
    end
    return table_concat(rows_out, "\n")
end
local table_gen =
    setmetatable(
    M,
    {__call = function(_, ...)
            return M.generate_table(...)
        end}
)
local get_script_name = function()
    local funca, err =
        pcall(
        function()
            genesis_error()
        end
    )
    return (not funca and err:match("\\(.*):(.*):") or nil)
end
local debug_print = function(text)
    client.color_log(83, 126, 242, string.format("[%s]\0", "genesis"))
    client.color_log(163, 163, 163, " ", text)
end
local debug_count = function(tab)
    local count = 0
    for _, _ in pairs(tab) do
        count = count + 1
    end
    return count
end
if package.plugin_tbc ~= nil then
    client.color_log(255, 0, 0, " - \0")
    client.color_log(255, 255, 255, "Failed to load \0")
    client.color_log(255, 0, 0, "[script is already active]")
    error()
end
local function clamp(min, max, value)
    if min < max then
        return math.min(max, math.max(min, value))
    else
        return math.min(min, math.max(max, value))
    end
end
local function get_lerp_time()
    local ud_rate = client.get_cvar("cl_updaterate")
    local min_ud_rate = client.get_cvar("sv_minupdaterate")
    local max_ud_rate = client.get_cvar("sv_maxupdaterate")
    if (min_ud_rate and max_ud_rate) then
        ud_rate = max_ud_rate
    end
    local ratio = client.get_cvar("cl_interp_ratio")
    if (ratio == 0) then
        ratio = 1
    end
    local lerp = client.get_cvar("cl_interp")
    local c_min_ratio = client.get_cvar("sv_client.min_interp_ratio")
    local c_max_ratio = client.get_cvar("sv_client.max_interp_ratio")
    if (c_min_ratio and c_max_ratio and c_min_ratio ~= 1) then
        ratio = clamp(ratio, c_min_ratio, c_max_ratio)
    end
    return math.max(lerp, (ratio / ud_rate))
end
local function is_record_valid(player_time, ms)
    local correct = 0
    correct = correct + get_lerp_time()
    correct = correct + client.latency()
    correct = clamp(correct, 0, ms)
    local delta = correct - (globals.curtime() - player_time)
    if math.abs(delta) > ms then
        return false
    end
    return true
end
ctoptions = {"tick correction", "fortify shot", "force fire", "fast charge"}
safeoptions = {"tick correction", "fortify shot"}
usoptions = {"fire on ticks", "speed boost"}
local script = {
    version = entity.get_player_name(entity.get_local_player()),
    debug = false,
    reference = {},
    interface = {
        ui.new_checkbox("RAGE", "Other", "Doubletap speed"),
        master_watermark_col = ui.new_color_picker("RAGE", "Other", "watermark_color", 100, 155, 255, 255),
        ui.new_slider("RAGE", "Other", "Tickbase", 16, 20, 18),
        ui.new_slider("RAGE", "Other", "States", -1, 2, 2, true, nil, 1, {[-1] = "Dynamic", [0] = "Stable"}),
        master_trigger = ui.new_multiselect("RAGE", "Other", "Trigger\ndt", ctoptions),
        ui.new_combobox("RAGE", "Other", "Watermark ui\ndt", {"Default", "New"}),
        ui.new_multiselect("RAGE", "Other", "unsafe\ndt", usoptions)
    }
}
ui.set(script.interface.master_trigger, {"tick correction", "fortify shot"})
function script:ui(name)
    local define = self.reference[name]
    if define == nil then
        error(string.format("unknown reference %s", name))
    end
    return {get_ids = function()
            return define[1]
        end, get_reffer = function()
            return define[2]
        end, call = function()
            local list = {}
            for i = 1, #define[1] do
                list[#list + 1] = ui.get(define[1][i])
            end
            return unpack(list)
        end, set = function(_, value, index, ignore_errors)
            local index = index or 1
            return ignore_errors == true and ({pcall(ui.set, define[1][index], value)})[1] or
                ui.set(define[1][index], value)
        end, set_cache = function(_, index, should_call, var)
            local index = index or 1
            if package._gcache == nil then
                package._gcache = {}
            end
            local name, _cond = tostring(define[1][index]), ui.get(define[1][index])
            local _type = type(_cond)
            local _, mode = ui.get(define[1][index])
            local finder = mode or (_type == "boolean" and tostring(_cond) or _cond)
            package._gcache[name] = package._gcache[name] or finder
            local hotkey_modes = {[0] = "always on", [1] = "on hotkey", [2] = "toggle", [3] = "off hotkey"}
            if should_call then
                ui.set(define[1][index], mode ~= nil and hotkey_modes[var] or var)
            else
                if package._gcache[name] ~= nil then
                    local _cache = package._gcache[name]
                    if _type == "boolean" then
                        if _cache == "true" then
                            _cache = true
                        end
                        if _cache == "false" then
                            _cache = false
                        end
                    end
                    ui.set(define[1][index], mode ~= nil and hotkey_modes[_cache] or _cache)
                    package._gcache[name] = nil
                end
            end
        end, set_visible = function(_, value, index)
            local index = index or 1
            ui.set_visible(define[1][index], value)
        end}
end
function script:ui_register(name, pdata)
    local ref_list = {}
    local ids = {pcall(ui.reference, unpack(pdata))}
    if ids[1] == false then
        error(string.format("%s cannot be defined (%s)", name, ids[2]))
    end
    if self.reference[name] ~= nil then
        error(string.format("%s is already taken in metatable", name))
    end
    for i = 2, #ids do
        ref_list[#ref_list + 1] = ids[i]
    end
    self.reference[name] = {ref_list, pdata}
    return self:ui(name)
end
local notes_pos = function(b)
    local c = function(d, e)
        local f = {}
        for g in pairs(d) do
            table.insert(f, g)
        end
        table.sort(f, e)
        local h = 0
        local i = function()
            h = h + 1
            if f[h] == nil then
                return nil
            else
                return f[h], d[f[h]]
            end
        end
        return i
    end
    local j = {get = function(k)
            local l, m = 0, {}
            for n, o in c(package.cnotes) do
                if o == true then
                    l = l + 1
                    m[#m + 1] = {n, l}
                end
            end
            for p = 1, #m do
                if m[p][1] == b then
                    return k(m[p][2] - 1)
                end
            end
        end, set_state = function(q)
            package.cnotes[b] = q
            table.sort(package.cnotes)
        end, unset = function()
            client.unset_event_callback("shutdown", callback)
        end}
    client.set_event_callback(
        "shutdown",
        function()
            if package.cnotes[b] ~= nil then
                package.cnotes[b] = nil
            end
        end
    )
    if package.cnotes == nil then
        package.cnotes = {}
    end
    return j
end
local fake_duck = ui.reference("RAGE", "Other", "Duck peek assist")
local ragebot, rkey = ui.reference("RAGE", "Aimbot", "Enabled")
local doubletap, doubletap_key = ui.reference("RAGE", "Other", "Double tap")
local g_client_log = function(text)
    client.color_log(135, 231, 255, "[genesis]\0")
    client.color_log(163, 163, 163, " ", text)
end
client.set_event_callback(
    "paint_ui",
    function()
        if login_return then
            return
        end
        if not login then
            username_ = "Ivan"
            g_client_log(string.format("welcome, %s | version: %s", username_, versions))
            if not playsound then
                client.exec("play UI/competitive_accept_beep.wav")
                playsound = true
            end
            allow_to_use = true
            login_return = true
            login = true
        end
    end
)
local function table_contains(table, item)
    for i = 1, #table do
        if table[i] == item then
            return true
        end
    end
    return false
end
local logs = {}
function addLog(text)
    table.insert(logs, {["text"] = text, ["time"] = globals.realtime()})
end
local backup_urmom = {fired_time = globals.curtime(), b_hitchance = nil}
local watermark_dragging = dragging.new("gnsdoubletap_watermark", 100, 200)
local dm = {}
local can_dt, can_expolitx, reset_hc = false, false, false
local ffi_cache = {}
local invoke_cache = function(b, c, d)
    local e = function(f, g, h)
        local i = {[0] = "always on", [1] = "on hotkey", [2] = "toggle", [3] = "off hotkey"}
        local j = tostring(f)
        local k = ui.get(f)
        local l = type(k)
        local m, n = ui.get(f)
        local o = n ~= nil and n or (l == "boolean" and tostring(k) or k)
        ffi_cache[j] = ffi_cache[j] or o
        if g then
            ui.set(f, n ~= nil and i[h] or h)
        else
            if ffi_cache[j] ~= nil then
                local p = ffi_cache[j]
                if l == "boolean" then
                    if p == "true" then
                        p = true
                    end
                    if p == "false" then
                        p = false
                    end
                end
                ui.set(f, n ~= nil and i[p] or p)
                ffi_cache[j] = nil
            end
        end
    end
    if type(b) == "table" then
        for q, r in pairs(b) do
            e(q, r[1], r[2])
        end
    else
        e(b, c, d)
    end
end
local initialization = function()
    local note = notes_pos "b_tbc.v2"
    package.plugin_tbc = true
    local hitchance = script:ui_register("hitchance", {"RAGE", "Aimbot", "Minimum hit chance"})
    local auto_fire = ui.reference("RAGE", "Aimbot", "Automatic fire")
    local dt_reserve = script:ui_register("dt_reserve", {"RAGE", "Other", "Double tap fake lag limit"})
    local usrcmd_maxpticks = script:ui_register("usrcmd_maxpticks", {"MISC", "Settings", "sv_maxusrcmdprocessticks"})
    local hold_aim = script:ui_register("hold_aim", {"MISC", "Settings", "sv_maxusrcmdprocessticks_holdaim"})
    local double_tap = script:ui_register("double_tap", {"RAGE", "Other", "Double tap"})
    local double_tap_mode = script:ui_register("double_tap_mode", {"RAGE", "Other", "Double tap mode"})
    local onshot_aa = script:ui_register("onshot_aa", {"AA", "Other", "On shot anti-aim"})
    local lowerbody = script:ui_register("lowerbody", {"AA", "Anti-aimbot angles", "Lower body yaw target"})
    local master_switch = script:ui_register("master_switch", {"RAGE", "Other", "Doubletap speed"})
    local master_ct_tickbase = script:ui_register("master_tickbase", {"RAGE", "Other", "Tickbase"})
    local master_ct_state_b = script:ui_register("master_ct_state_b", {"RAGE", "Other", "States"})
    local master_watermark = script:ui_register("master_watermark", {"RAGE", "Other", "Watermark ui\ndt"})
    local master_unsafe = script:ui_register("master_unsafe", {"RAGE", "Other", "unsafe\ndt"})
    local ucmd_maxpticks = ui.reference("MISC", "Settings", "sv_maxusrcmdprocessticks")
    local is_knife, is_1bulltet, is_auto, is_pistols, is_rifles = false, false, false, false, false
    local clock = cvar.cl_clock_correction
    local d_delay = 0
    local cmove = {
        old_tickbase = 0,
        old_sim_time = 0,
        old_command_num = 0,
        skip_next_differ = false,
        charged_before = false,
        did_shift_before = false,
        can_shift_tickbase = 0,
        is_cmd_safe = true,
        last_charge = 0,
        validate_cmd = usrcmd_maxpticks:call(),
        lag_state = nil
    }
    local caimbot = {data = {}, shift_time = 0, shift_data = {}}
    local reset = function()
        if cmove.lag_state ~= nil then
            if script.debug then
                if table_contains(ui.get(script.interface.master_trigger), ctoptions[999]) then
                    double_tap:set(cmove.lag_state)
                    cmove.lag_state = nil
                end
            end
        end
        for i in pairs(script.reference) do
            local element = script:ui(i)
            for j = 1, #element:get_ids() do
                element:set_cache(j, false)
            end
        end
    end
    local g_doubletap_controller = function(e)
        local next_shift_amount = 0
        local should_break_tbc = false
        local me = entity.get_local_player()
        local wpn = entity.get_player_weapon(me)
        local wpn_name = entity.get_classname(wpn) or ""
        local wpn_id = entity.get_prop(wpn, "m_iItemDefinitionIndex")
        local m_item = wpn_id and bit.band(wpn_id, 0xFFFF) or 0
        local can_exploit = function(me, wpn, ticks_to_shift)
            if wpn == nil then
                return false
            end
            local tickbase = entity.get_prop(me, "m_nTickBase")
            local curtime = globals.tickinterval() * (tickbase - ticks_to_shift)
            if not allow_to_use then
                return false
            end
            if not master_switch:call() then
                return false
            end
            if ui.get(fake_duck) then
                return false
            end
            if curtime < entity.get_prop(me, "m_flNextAttack") then
                return false
            end
            if curtime < entity.get_prop(wpn, "m_flNextPrimaryAttack") then
                return false
            end
            return true
        end
        if cmove.validate_cmd > 0 then
            cmove.validate_cmd = cmove.validate_cmd - 1
            local dt, dt_key = double_tap:call()
            if dt and dt_key then
                should_break_tbc = true
            end
        end
        ::begin_command::
        local ready_to_shift = can_exploit(me, wpn, 13)
        local weapon_ready = can_exploit(me, wpn, math.abs(-1 - next_shift_amount))
        can_expolitx = can_exploit(me, wpn, 13)
        if ready_to_shift == true or weapon_ready == false and cmove.did_shift_before == true then
            next_shift_amount = 13
        else
            next_shift_amount = 0
        end
        local tickbase = entity.get_prop(me, "m_nTickBase")
        if cmove.old_tickbase ~= 0 and tickbase < cmove.old_tickbase then
            if cmove.old_tickbase - tickbase > 11 then
                cmove.skip_next_differ = true
                cmove.charged_before = false
                cmove.can_shift_tickbase = false
            end
        end
        local difference = e.command_number - cmove.old_command_num
        if difference >= 11 and difference <= usrcmd_maxpticks:call() then
            cmove.can_shift_tickbase = not cmove.skip_next_differ
            cmove.charged_before = cmove.can_shift_tickbase
            cmove.last_charge = difference + 1
            cmove.is_cmd_safe = difference > 3 and math.abs(usrcmd_maxpticks:call() - difference) <= 3
            d_delay = math.abs(usrcmd_maxpticks:call() - cmove.last_charge)
            addLog(string.format("charged: %s (%s)", difference + 1, cmove.is_cmd_safe and "safe" or "unsafe"))
        end
        if ready_to_shift == false then
            cmove.can_shift_tickbase = false
        else
            cmove.can_shift_tickbase = cmove.charged_before
        end
        cmove.old_tickbase = tickbase
        cmove.old_command_num = e.command_number
        cmove.skip_next_differ = false
        cmove.did_shift_before = next_shift_amount ~= 0
        if table_contains(ui.get(script.interface.master_trigger), ctoptions[4]) then
            cmove.can_shift_tickbase = cmove.can_shift_tickbase and 2 or 0
        else
            if master_ct_state_b:call() == -1 then
                cmove.can_shift_tickbase = cmove.can_shift_tickbase and 2 or 1
            elseif master_ct_state_b:call() == 0 then
                cmove.can_shift_tickbase = cmove.can_shift_tickbase and 2 or 0
            elseif master_ct_state_b:call() > 0 then
                cmove.can_shift_tickbase = master_ct_state_b:call()
            end
        end
        if cmove.can_shift_tickbase == 0 and cmove.charged_before == true then
            cmove.can_shift_tickbase = 1
        end
        if cmove.can_shift_tickbase == 0 then
            cmove.last_charge = 0
        end
    end
    local g_aimbot_listener = function(e)
        local run_qm = false
        local is_inactive = caimbot.shift_time == 0
        backup_urmom.fired_time = globals.curtime()
        if double_tap_mode:call() == "Offensive" and is_inactive and cmove.can_shift_tickbase == 2 then
            caimbot.shift_time = 1
            caimbot.data[debug_count(caimbot.data) + 1] = {e.x, e.y, e.z}
            run_qm = true
        end
        if
            table_contains(master_unsafe:call(), usoptions[2]) and double_tap_mode:call() == "Offensive" and
                script.debug and
                status == "debug"
         then
            if is_auto then
                hitchance:set_cache(1, run_qm, 0)
            end
        end
    end
    local g_command_controller = function(e)
        if cmove.lag_state ~= nil then
            if script.debug then
                if table_contains(ui.get(script.interface.master_trigger), ctoptions[999]) then
                    if cmove.last_charge < 15 and can_dt then
                        double_tap:set(cmove.lag_state)
                        cmove.lag_state = nil
                    end
                end
            end
        end
        local osa, osa_key = onshot_aa:call()
        local dt, dt_key = double_tap:call()
        local cs_tickbase = cmove.can_shift_tickbase
        ::begin_command::
        local me = entity.get_local_player()
        local losc = dt and dt_key and double_tap_mode:call() == "Offensive"
        local fired_this_tick = false
        local reset_command = false
        local m_vecvel = {entity.get_prop(me, "m_vecVelocity")}
        local velocity = math.floor(math.sqrt(m_vecvel[1] ^ 2 + m_vecvel[2] ^ 2 + m_vecvel[3] ^ 2) + 0.5)
        if caimbot.shift_time > 0 and can_dt then
            local current_command = e
            local max_commands = cmove.last_charge
            local aimbot_command = caimbot.data[debug_count(caimbot.data)]
            if
                debug_count(caimbot.data) > 0 and aimbot_command ~= nil and
                    table_contains(ui.get(script.interface.master_trigger), ctoptions[2])
             then
                local eye_pos = vector(client.eye_position())
                local fire_vector = vector(unpack(aimbot_command))
                local entindex, dmg = eye_pos:trace_bullet_to(fire_vector, me, true)
                local aim_at = eye_pos:angle_to(fire_vector)
                if table_contains(ui.get(script.interface.master_trigger), ctoptions[3]) then
                    e.in_attack = 1
                    if (caimbot.shift_time == max_commands or max_commands < 1) and e.in_attack == 0 then
                        addLog("force fired")
                        e.in_attack = 1
                    end
                else
                    if caimbot.shift_time == max_commands or max_commands < 1 then
                        if dmg <= 0 then
                            e.in_attack = 0
                            addLog("skipped shift")
                        else
                            fired_this_tick = true
                            reset_command = true
                        end
                    end
                end
            end
            caimbot.shift_data[#caimbot.shift_data + 1] = {
                caimbot.shift_time,
                cs_tickbase,
                e.chokedcommands,
                entity.get_prop(me, "m_nTickBase"),
                globals.tickcount(),
                "false"
            }
            if
                caimbot.shift_time ~= 0 and
                    (reset_command == true or caimbot.shift_time == max_commands or max_commands < 1)
             then
                caimbot.shift_time = 0
                caimbot.shift_data = {}
                caimbot.data = {}
            else
                caimbot.shift_time = caimbot.shift_time + 1
            end
        end
        if
            (caimbot.shift_time == 0 and fired_this_tick == false) and
                ((losc == true and cs_tickbase == 0) or (velocity <= 1 and cs_tickbase == 2))
         then
            cmove.lag_state = dt
            if debug_count(caimbot.shift_data) > 0 then
                caimbot.shift_data[debug_count(caimbot.shift_data)][6] = tostring(dt)
            end
        end
        if
            table_contains(master_unsafe:call(), usoptions[1]) and double_tap_mode:call() == "Offensive" and
                script.debug and
                status == "debug"
         then
            local unknownx = caimbot.shift_time > 0 or fired_this_tick
            local unknowx = can_dt and is_auto
            hitchance:set_cache(1, unknownx and unknowx, 0)
        end
        if cmove.lag_state ~= nil then
            if script.debug then
                if table_contains(ui.get(script.interface.master_trigger), ctoptions[999]) then
                    if cmove.last_charge < 15 and can_dt then
                        double_tap:set(false)
                    end
                end
            end
        end
    end
    local g_command_run = function(e)
        if cmove.lag_state ~= nil then
            if script.debug then
                if table_contains(ui.get(script.interface.master_trigger), ctoptions[999]) then
                    if cmove.last_charge < 15 and can_dt then
                        double_tap:set(cmove.lag_state)
                        cmove.lag_state = nil
                    end
                end
            end
        end
        dt_reserve:set_cache(1, true, 1)
        usrcmd_maxpticks:set_cache(1, true, master_ct_tickbase:call())
        if ui.get(ucmd_maxpticks) ~= master_ct_tickbase:call() then
            ui.set(ucmd_maxpticks, master_ct_tickbase:call())
        end
    end
    local GLOBAL_ALPHA, GLOBAL_ALPHA_2 = 0, 0
    local xscreen_x, xscreen_y = client.screen_size()
    local function is_dragging_menu()
        local x, y = ui.mouse_position()
        local px, py = ui.menu_position()
        local sx, sy = ui.menu_size()
        local click = client.key_state(0x01)
        local in_x = x > px and x < px + sx
        local in_y = y > py and y < py + sy
        return in_x and in_y and click and ui.is_menu_open()
    end
    local dl = 1
    local function ui_wm_left(x, y, w, r, g, b, alpha, ga, text)
        if master_watermark:call() == "New" then
            renderer.rectangle(x + 6, y, w - 2, 21, 17, 17, 17, 150)
            renderer.triangle((x - 5) + 5, y + 8, x + 6, y + 8, x + 6, y, 17, 17, 17, 150)
            renderer.rectangle((x - 5) + 5, y + 8, 6, 13, 17, 17, 17, 150)
            renderer.line((x - 5) + 5, y + 8, (x + 2) + 5, y, r, g, b, 255)
            renderer.rectangle((x - 5) + 5, y + 8, 1, 13, r, g, b, 255)
            renderer.rectangle(x + 6, y, w - 2, 1, r, g, b, 255)
            renderer.text(x + 6, y + 5, 255, 255, 255, 255, "", 0, text)
        else
            renderer.rectangle(x - 1, y, w + 2, 21, 17, 17, 17, 150)
            renderer.gradient(x - 1, y + 21, 1.7, -ga * 21, r, g, b, 255, r, g, b, alpha, false)
            renderer.gradient(x + w, y + 21, 1.7, -ga * 21, r, g, b, 255, r, g, b, alpha, false)
            renderer.gradient(x - 1, y + 21, ga * w / 2 + 1.5, 1.7, r, g, b, 255, r, g, b, alpha, true)
            renderer.gradient(x + w + 1, y + 21, -ga * w / 2 - 1, 1.7, r, g, b, 255, r, g, b, alpha, true)
            renderer.gradient(x + w / 2, y, -ga * w / 2 - 1, 1.7, r, g, b, 255, r, g, b, alpha, true)
            renderer.gradient(x + w / 2, y, ga * w / 2 + 1, 1.7, r, g, b, 255, r, g, b, alpha, true)
            renderer.text(x + 3, y + 5, 255, 255, 255, 255, "", 0, text)
        end
    end
    local xRetard = 0
    local function ui_wm_right(x, y, w, r, g, b, alpha, ga, text)
        if master_watermark:call() == "New" then
            renderer.rectangle(x - w + 6, y, w - 2, 21, 17, 17, 17, 150)
            renderer.triangle((x - w - 5) + 5, y + 8, x - w + 6, y + 8, x - w + 6, y, 17, 17, 17, 150)
            renderer.rectangle((x - w - 5) + 5, y + 8, 6, 13, 17, 17, 17, 150)
            renderer.line((x - w - 5) + 5, y + 8, (x - w + 2) + 5, y, r, g, b, 255)
            renderer.rectangle((x - w - 5) + 5, y + 8, 1, 13, r, g, b, 255)
            renderer.rectangle(x - w + 6, y, w - 2, 1, r, g, b, 255)
            renderer.text(x - w + 6, y + 5, 255, 255, 255, 255, "", 0, text)
        else
            renderer.rectangle(x - w + 6, y, w - 2 + 2, 21, 17, 17, 17, 150)
            renderer.gradient(x - w + 6, y + 21, 1.7, -ga * 21, r, g, b, 255, r, g, b, alpha, false)
            renderer.gradient(x + 6, y + 21, 1.7, -ga * 21, r, g, b, 255, r, g, b, alpha, false)
            renderer.gradient(x - w + 6, y + 21, ga * w / 2 + 0.5, 1.7, r, g, b, 255, r, g, b, alpha, true)
            renderer.gradient(x + 7, y + 21, -ga * w / 2 - 1, 1.7, r, g, b, 255, r, g, b, alpha, true)
            renderer.gradient(x - w / 2 + 6, y, -ga * w / 2, 1.7, r, g, b, 255, r, g, b, alpha, true)
            renderer.gradient(x - w / 2 + 6, y, ga * w / 2 + 1, 1.7, r, g, b, 255, r, g, b, alpha, true)
            renderer.text(x - w + 10, y + 5, 255, 255, 255, 255, "", 0, text)
        end
        xRetard = x - w + 10
    end
    function clamp_alpha(...)
        local s = {...}
        table.sort(s)
        return s[2]
    end
    local function watermark()
        local realtime = globals.realtime() % 3
        local alpha = math.floor(math.sin(realtime * 4) * (255 / 2 - 1) + 255 / 2)
        local dy = (can_dt and 4 or 6) * globals.frametime()
        if is_knife or not entity.is_alive(entity.get_local_player()) then
            GLOBAL_ALPHA = GLOBAL_ALPHA + dy
            if GLOBAL_ALPHA > 1 then
                GLOBAL_ALPHA = 1
            end
        else
            if can_expolitx then
                GLOBAL_ALPHA = GLOBAL_ALPHA + dy
                if GLOBAL_ALPHA > 1 then
                    GLOBAL_ALPHA = 1
                end
            else
                GLOBAL_ALPHA = GLOBAL_ALPHA - dy
                if GLOBAL_ALPHA <= 0 then
                    GLOBAL_ALPHA = 0
                end
            end
        end
        local text, tickbases, states =
            "",
            can_expolitx and cmove.last_charge or 0,
            can_expolitx and cmove.can_shift_tickbase or 0
        text = text .. string.format("Genesis")
        if status ~= "default" then
            text = text .. string.format(" [%s]", status == "beta" and "alpha" or status)
        end
        text = text .. string.format(" | %s", username_)
        if can_dt and not is_knife and not is_1bulltet and entity.is_alive(entity.get_local_player()) then
            text = text .. string.format(" | tickbase: %s/%s", tickbases, states)
            text = text .. string.format("(%s", can_expolitx and d_delay or "-")
            text = text .. string.format("%s", can_expolitx and "%)" or ")")
            if cmove.is_cmd_safe == false then
                text = text .. string.format(" | safe: %s", cmove.is_cmd_safe)
            end
        end
        local w, h = renderer.measure_text("", text) + 7
        local x, y = watermark_dragging:get()
        local r, g, b, a = ui.get(script.interface.master_watermark_col)
        if x < xscreen_x / 2 then
            ui_wm_left(x + 3, y, w, r, g, b, alpha, GLOBAL_ALPHA, text)
        else
            ui_wm_right(x + 210, y, w, r, g, b, alpha, GLOBAL_ALPHA, text)
        end
        if ui.is_menu_open() and not is_dragging_menu() then
            watermark_dragging:drag(x < xscreen_x / 2 and w or w + 80, 37)
        end
        local color = {
            math.floor(math.sin(globals.realtime() * 2) * 127 + 128),
            math.floor(math.sin(globals.realtime() * 2 + 2) * 127 + 128),
            math.floor(math.sin(globals.realtime() * 2 + 4) * 127 + 128)
        }
        local wpn = entity.get_player_weapon(entity.get_local_player())
        local wpn_id = entity.get_prop(wpn, "m_iItemDefinitionIndex")
        local weapons = wpn_id and bit.band(wpn_id, 0xFFFF) or 0
        if weapons == nil or not entity.is_alive(entity.get_local_player()) then
            return
        end
        local icon_width, icon_width_2 = 0, 0
        bullet = {
            32,
            32,
            '<?xml version="1.0" encoding="utf-8"?><svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="32" height="32"><g><g><path fill="#FFFFFF" d="M5,24.5l5.4,4.3l9.8-13c0.2-0.3,0.5-1.2,0.5-1.2s0.1-0.7,0.5-1.4c0.4-0.7,1.2-1.7,1.2-1.7l-3.7-3.1c0,0-1.6,1.5-1.8,1.7c-0.7,0.7-1.5,1-1.8,1.4S5,24.5,5,24.5z"/><polygon fill="#FFFFFF" points="19.3,7.8 23,10.8 27.9,1.8 26.6,0.7"/><path fill="#FFFFFF" d="M4.4,25.2l5.5,4.4l-0.5,0.5c0,0-2,0-3.7-1.3c-1.9-1.4-1.7-3.1-1.7-3.1L4.4,25.2z"/></g></g></svg>'
        }
        local icon = images.get_weapon_icon(weapons)
        local svg = renderer.load_svg(bullet[3], 32 * 0.6, 25 * 0.6)
        if icon == nil then
            return
        end
        icon_width = icon:measure(nil, 14)
        icon:draw(x < xscreen_x / 2 and x + 5 or xRetard, y + 25, nil, 16, 255, 255, 255, 255, true, "f")
        if can_dt then
            if weapons == 7 or weapons == 39 or weapons == 16 then
                dl = 3
            elseif
                weapons == 1 or weapons == 30 or weapons == 36 or weapons == 2 or weapons == 4 or weapons == 61 or
                    weapons == 11 or
                    weapons == 38 or
                    weapons == 32
             then
                dl = 2
            elseif
                weapons == 34 or weapons == 19 or weapons == 25 or weapons == 35 or weapons == 29 or weapons == 14 or
                    weapons == 28 or
                    weapons == 27
             then
                dl = 5
            elseif
                weapons == 33 or weapons == 24 or weapons == 26 or weapons == 17 or weapons == 13 or weapons == 10 or
                    weapons == 60 or
                    weapons == 8
             then
                dl = 4
            elseif weapons == 9 or weapons == 31 or weapons == 40 or is_knife then
                dl = 1
            end
        else
            dl = 1
        end
        local x_gy, x_ge = 0, 0
        for am = 1, dl do
            local cV = can_expolitx
            if dm[am] == nil then
                dm[am] = 0
            end
            if cV and dm[1] < 1 then
                dm[1] = dm[1] + dy
                if dm[1] > 1 then
                    dm[1] = 1
                end
            elseif cV and dm[am] == 1 and dm[am] < dm[1] then
                dm[am] = dm[am] + dy
                if dm[am] > 1 then
                    dm[am] = 1
                end
            elseif cV and dm[am - 1] == 1 and dm[am] < dm[am - 1] then
                dm[am] = dm[am] + dy
                if dm[am] > 1 then
                    dm[am] = 1
                end
            elseif not cV then
                if dm[am] == nil then
                    dm[am] = 0
                end
                dm[am] = dm[am] - dy
                if dm[am] <= 0 then
                    dm[am] = 0
                end
            end
            local dG = 13 * am
            x_gy = x < xscreen_x / 2 and x + icon_width + dG or xRetard + icon_width + dG
            if dl == 1 then
                x_ge = is_knife and dm[am] * 5 or GLOBAL_ALPHA * dm[am] * 20
            elseif dl == 2 then
                x_ge = GLOBAL_ALPHA * dm[am] * 30
            elseif dl == 3 then
                x_ge = GLOBAL_ALPHA * dm[am] * 40
            elseif dl == 4 then
                x_ge = GLOBAL_ALPHA * dm[am] * 50
            elseif dl == 5 then
                x_ge = GLOBAL_ALPHA * dm[am] * 70
            end
            if not is_knife then
                renderer.texture(
                    svg,
                    x < xscreen_x / 2 and x + icon_width + dG or xRetard + icon_width + dG,
                    y + 25,
                    32 * 0.6,
                    25 * 0.6,
                    255,
                    255,
                    255,
                    (can_expolitx and cmove.is_cmd_safe == false and can_dt) and GLOBAL_ALPHA * dm[am] * alpha or
                        GLOBAL_ALPHA * dm[am] * 255
                )
            end
        end
        for i = 1, #logs do
            if globals.realtime() - logs[i].time > 5 or i > 6 then
                if i > 6 then
                    table.remove(logs, i - 5)
                else
                    table.remove(logs, i)
                end
                return
            end
            local anim_time = globals.realtime() - logs[i].time
            local fadein_out = clamp_alpha(0, 255 * (anim_time), 255)
            if anim_time >= 4 then
                fadein_out = math.abs(clamp_alpha(0, 255 * (anim_time - 4), 255) - 255)
            end
            renderer.text(
                ((x < xscreen_x / 2 and x or xRetard) + icon_width + x_ge) + 20,
                y + 12 + (i * 12),
                255,
                255,
                255,
                fadein_out,
                "",
                nil,
                logs[i].text
            )
        end
    end
    local g_paint_handler = function(ctx)
        note.set_state(false)
        local realtime = globals.realtime() % 3
        local alpha = math.floor(math.sin(realtime * 4) * (255 / 2 - 1) + 255 / 2)
        if not allow_to_use then
            client.draw_gradient(ctx, 0, 0, xscreen_x, xscreen_y, 0, 0, 0, 255, 0, 0, 0, 255, false)
            client.draw_text(ctx, xscreen_x / 2, xscreen_y / 2, 200, 200, 200, alpha, "+", 0, "WHO.RU")
            return
        end
        if entity.get_local_player() then
            watermark()
        end
        clock:set_int(table_contains(ui.get(script.interface.master_trigger), ctoptions[1]) and 0 or 1)
        if entity.is_alive(entity.get_local_player()) == false or not master_switch:call() then
            return reset()
        end
    end
    client.set_event_callback(
        "weapon_fire",
        function(c)
            local me = entity.get_local_player()
            local user = client.userid_to_entindex(c.userid)
            if me == user then
                if can_dt and not is_knife and not client.key_state(0x01) then
                    if caimbot.shift_time > 5 and math.floor(client.latency() * 1000 + 0.5) > 1 then
                        addLog(
                            string.format(
                                "delay: %s (%sms)",
                                caimbot.shift_time,
                                math.floor(client.latency() * 1000 + 0.5)
                            )
                        )
                    end
                end
            end
        end
    )
    hideshot, hideshotkey = ui.reference("AA", "Other", "On shot anti-aim")
    local backup_hc = function()
        local osa, osa_key = onshot_aa:call()
        local hideshots = ui.get(hideshot) and ui.get(hideshotkey)
        local shots =
            (backup_urmom.fired_time ~= nil and can_dt and not hideshots and
            (is_record_valid(backup_urmom.fired_time, 0.5))) and
            is_auto
        if is_auto and backup_urmom.b_hitchance == nil and hitchance:call() ~= 0 then
            backup_urmom.b_hitchance = hitchance:call()
        end
        if backup_urmom.b_hitchance ~= nil and is_auto then
            if
                backup_urmom.b_hitchance ~= hitchance:call() and hitchance:call() ~= 0 and not shots and
                    backup_urmom.b_hitchance ~= nil
             then
                backup_urmom.b_hitchance = hitchance:call()
            end
            if shots then
            else
                if
                    hitchance:call() ~= backup_urmom.b_hitchance and hitchance:call() == 0 and
                        backup_urmom.b_hitchance ~= nil
                 then
                    hitchance:set_cache(1, true, backup_urmom.b_hitchance)
                end
            end
        end
    end
    local doubletap = function()
        if ui.is_menu_open() then
            dt_reserve:set_visible(not master_switch:call())
            allowyes = allow_to_use and master_switch:call()
            master_switch:set_visible(allow_to_use, 1)
            master_watermark:set_visible(allowyes, 1)
            ui.set_visible(script.interface.master_trigger, allowyes)
            master_unsafe:set_visible(allowyes and double_tap_mode:call() == "Offensive" and script.debug, 1)
            master_ct_tickbase:set_visible(allowyes, 1)
            master_ct_state_b:set_visible(
                allowyes and not table_contains(ui.get(script.interface.master_trigger), ctoptions[4]),
                1
            )
        end
        if
            table_contains(master_unsafe:call(), usoptions[2]) and double_tap_mode:call() == "Offensive" and
                script.debug and
                status == "debug"
         then
            backup_hc()
        end
        can_dt = ui.get(doubletap) and ui.get(doubletap_key) and not ui.get(fake_duck)
        if dt_reserve:call() ~= 1 and not master_switch:call() then
            dt_reserve:set_cache(1, true, 1)
        end
        if not entity.is_alive(entity.get_local_player()) then
            return
        end
        local wpn_id = entity.get_prop(entity.get_player_weapon(entity.get_local_player()), "m_iItemDefinitionIndex")
        local weapons = wpn_id ~= nil and bit.band(wpn_id, 0xFFFF) or 0
        if weapons == nil or wpn_id == nil then
            return
        end
        is_knife =
            wpn_id >= 500 and wpn_id <= 525 or wpn_id == 41 or wpn_id == 42 or wpn_id == 59 or
            wpn_id >= 43 and wpn_id <= 49
        is_1bulltet = wpn_id == 9 or wpn_id == 31 or wpn_id == 40
        is_auto = wpn_id == 11 or wpn_id == 38
        is_pistols =
            wpn_id >= 1 and wpn_id <= 4 or wpn_id == 30 or wpn_id == 32 or wpn_id == 36 or wpn_id == 61 or wpn_id == 63
        is_rifles = wpn_id == 7 or wpn_id == 10 or wpn_id == 13 or wpn_id == 16 or wpn_id == 39
    end
    client.set_event_callback(
        "shutdown",
        function()
            dt_reserve:set_visible(true)
            dt_reserve:set_cache(1, true, 1)
            ui.set(ucmd_maxpticks, 16)
            clock:set_int(1)
        end
    )
    client.set_event_callback("predict_command", g_doubletap_controller)
    client.set_event_callback("setup_command", g_command_controller)
    client.set_event_callback("run_command", g_command_run)
    client.set_event_callback("aim_fire", g_aimbot_listener)
    client.set_event_callback("paint_ui", g_paint_handler)
    client.set_event_callback("shutdown", reset)
    client.set_event_callback("paint_ui", doubletap)
end
initialization()
