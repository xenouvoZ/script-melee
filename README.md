--By Rufus14 (fps unlocker recommended even more)
local tweenservice = game:GetService("TweenService")
local runservice = game:GetService("RunService")
local debris = game:GetService("Debris")
local players = game:GetService("Players")
local tool = script.Parent
local owner
local charhum
local character
local charroot
local chartorso
local charhead
local remote = tool.fistremote
local easingstyles = Enum.EasingStyle
local easingdirs = Enum.EasingDirection

--data
local DAMAGE = 20
local counterDAMAGE = 10
local currentwelds = {} --dont repeat names
local currentsounds = {} --dont repeat names
local currenttrails = {}
local currentattachs = {}
local lastattachpos = {}
local ignoretable = {}
local peoplehit = {}
local toolstate = "idle"
local attachmentname = "ro bIox"
local counterdetectionMAG = 8
local swinganimation = 3
local lookpartheight = 0.2
local leftkickcooldown = 1.5
local rightkickcooldown = 1
local counterFAILEDcooldown = 5
local blockwsPERCENTAGE = 0.7 --this is how much % it will reduce your speed when blocking
local kickwsPERCENTAGE = 0.85 --this is how much % it will reduce your speed when kicking
local counterwsPERCENTAGE = 0.9 --this is how much % it will reduce your speed when countering
local cankick = true
local cancounter = true
local runfunc
local currentanimtick
local currentequiptick

local swingdatas = { --some swings are just too slow
	[1] = {"Right Arm", 0.28},
	[2] = {"Left Arm", 0.28},
	[3] = {"Right Arm", 0.3},
	[4] = {"Left Arm", 0.3}
}
local hitvector = { --which way the character spins when they get hit
	["Right Arm"] = 1,
	["Left Arm"] = -1
}
local illegalstates = {
	"idle",
	"block",
	"counter"
}

local sfxdata = { --dont repeat names (id, vol, rollmin, rollmax, playonremove)
	["punch1"] = {"4306980885", 1.5, 5, 60, true},
	["punch2"] = {"3932505023", 1.5, 5, 60, true},
	["punch3"] = {"3932506183", 1.5, 5, 60, true},
	["punch4"] = {"3932504231", 1.5, 5, 60, true},
	["punch5"] = {"4306980885", 1.5, 5, 60, true},
	["bash"] = {"2801263", 1, 5, 60, true},
	["concrete"] = {"1476374050", 0.5, 5, 60, true},
	["block1"] = {"4306994664", 2, 5, 60, true},
	["block2"] = {"4306995205", 2, 5, 60, true},
	["block3"] = {"4306994923", 2, 5, 60, true},
	["grapple1"] = {"4086209510", 2, 5, 60, true},
	["grapple2"] = {"4086209719", 2, 5, 60, true},
	["AUGHHHH"] = {"8627655583", 1, 2, 60, true},
	["fabric"] = {"9114890978", 1, 5, 80, true},
	["swing1"] = {"8907343824", 1, 5, 80, true},
	["swing2"] = {"9076453292", 1, 5, 80, true},
	["knuckles"] = {"691183830", 4, 0.2, 60},
	["hitflesh1"] = {"3744371091", 3, 2, 60, true},
	["hitflesh2"] = {"3744371864", 3, 2, 60, true},
	["hitflesh3"] = {"3744371342", 3, 2, 60, true},
	["bone"] = {"4086172420", 1, 3, 60, true},
	["snap"] = {"4086190876", 1, 3, 60, true},
	["equip"] = {"4549835866", 2, 1, 60},
}
local sfxbehavior = {
	["punch1"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 3
		eq.HighGain = 3
	end,
	["punch2"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 3
		eq.HighGain = 3
	end,
	["punch3"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 3
		eq.HighGain = 3
	end,
	["punch4"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 3
		eq.HighGain = 3
	end,
	["punch5"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 3
		eq.HighGain = 3
	end,
	["hitflesh1"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 3
		eq.HighGain = 3
	end,
	["hitflesh2"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 3
		eq.HighGain = 3
	end,
	["hitflesh3"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 3
		eq.HighGain = 3
	end,
	["AUGHHHH"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 3
		eq.HighGain = 3
	end,
	["concrete"] = function(so)
		local eq = Instance.new("EqualizerSoundEffect", so)
		eq.LowGain = 10
		eq.MidGain = 0
		eq.HighGain = 0
	end,
}
local attachmentdata = {
	["Right Arm"] = {repeatY = 3, depth = 1, placement = {
		Vector3.new(0,-1,0),

		Vector3.new(0,-1,0.5),
		Vector3.new(0,-1,-0.5),

		Vector3.new(0.5,-1,0),
		Vector3.new(-0.5,-1,0),

		Vector3.new(0.5,-1,0.5),
		Vector3.new(-0.5,-1,-0.5),

		Vector3.new(-0.5,-1,0.5),
		Vector3.new(0.5,-1,-0.5),
	}},
	["Left Arm"] = {repeatY = 3, depth = 1, placement = {
		Vector3.new(0,-1,0),

		Vector3.new(0,-1,0.5),
		Vector3.new(0,-1,-0.5),

		Vector3.new(0.5,-1,0),
		Vector3.new(-0.5,-1,0),

		Vector3.new(0.5,-1,0.5),
		Vector3.new(-0.5,-1,-0.5),

		Vector3.new(-0.5,-1,0.5),
		Vector3.new(0.5,-1,-0.5),
	}},
	["Left Leg"] = {repeatY = 3, depth = 1, placement = {
		Vector3.new(0,-1,0),

		Vector3.new(0,-1,0.5),
		Vector3.new(0,-1,-0.5),

		Vector3.new(0.5,-1,0),
		Vector3.new(-0.5,-1,0),

		Vector3.new(0.5,-1,0.5),
		Vector3.new(-0.5,-1,-0.5),

		Vector3.new(-0.5,-1,0.5),
		Vector3.new(0.5,-1,-0.5),
	}},
	["Right Leg"] = {repeatY = 3, depth = 1, placement = {
		Vector3.new(0,-1,0),

		Vector3.new(0,-1,0.5),
		Vector3.new(0,-1,-0.5),

		Vector3.new(0.5,-1,0),
		Vector3.new(-0.5,-1,0),

		Vector3.new(0.5,-1,0.5),
		Vector3.new(-0.5,-1,-0.5),

		Vector3.new(-0.5,-1,0.5),
		Vector3.new(0.5,-1,-0.5),
	}},
}
local animrawdata = { --had to :inverse() rootpart on every keyframe because i put torso as part0 for no literal reason
	["equip"] = {
		[1] = {
			["rarmweld"] = {CFrame.new(1.80360985, -0.832960367, -0.48553133, 0.344310164, -0.319109023, 0.882960677, 0.938373387, 0.0868239701, -0.334539413, 0.030092366, 0.94373244, 0.329337776), easingstyles.Sine, easingdirs.In, 0.1},
			["larmweld"] = {CFrame.new(-1.74353695, -0.8067801, -0.516905785, 0.351057112, 0.302011251, -0.886311531, -0.936352611, 0.111618757, -0.332843632, -0.00159355253, 0.946747243, 0.321973592), easingstyles.Sine, easingdirs.In, 0.1},
			["headtorootweld"] = {CFrame.new(0, 0.300000191, 0, 1, -1.20370622e-35, 0, -1.20370622e-35, 1, -4.21297175e-35, 0, -4.21297175e-35, 1), easingstyles.Sine, easingdirs.In, 0.1},
			["rootweld"] = {CFrame.new(), easingstyles.Sine, easingdirs.InOut, 0.2},
		},
		[2] = {
			["rarmweld"] = {CFrame.new(0.802461624, -0.623572588, -0.795546055, 0.512912929, 0.845280886, 0.149733871, 0.858436882, -0.505547106, -0.0866472796, 0.002456218, 0.172979593, -0.98492229), easingstyles.Quart, easingdirs.Out, 0.3},
			["larmweld"] = {CFrame.new(-0.471618652, -0.927988291, -0.812986135, 0.198409885, -0.971196473, -0.13194859, -0.976539075, -0.184390634, -0.111222036, 0.0836883634, 0.150920734, -0.984996915), easingstyles.Quart, easingdirs.Out, 0.3},
			["headtorootweld"] = {CFrame.new(1.90734863e-06, 0.300001621, 4.76837158e-07, 0.999999821, 8.38190317e-08, -7.4505806e-08, 7.4505806e-08, 0.999999344, 2.23517418e-08, -5.96046448e-08, 2.23517418e-08, 0.999999821), easingstyles.Quart, easingdirs.Out, 0.3},
			["rootweld"] = {CFrame.new(), easingstyles.Sine, easingdirs.InOut, 0.2},
		},
		[3] = {
			["rarmweld"] = {CFrame.new(0.687065125, -0.664569378, -0.809481382, 0.358339161, 0.921505272, 0.149733886, 0.933182001, -0.348800361, -0.0866472423, -0.0276186578, 0.170778066, -0.984922051), easingstyles.Elastic, easingdirs.Out, 0.4},
			["larmweld"] = {CFrame.new(-0.375936508, -1.00558329, -0.817040682, 0.026749067, -0.990895391, -0.131948605, -0.99372226, -0.0120151229, -0.111222036, 0.108624034, 0.13409555, -0.984996855), easingstyles.Elastic, easingdirs.Out, 0.4},
			["headtorootweld"] = {CFrame.new(1.90734863e-06, 0.300002575, 9.53674316e-07, 0.999999821, 1.05239451e-07, -5.96046448e-08, 9.49949026e-08, 0.999999285, 3.7252903e-08, -2.98023224e-08, 3.7252903e-08, 0.999999762), easingstyles.Elastic, easingdirs.Out, 0.4},
			["rootweld"] = {CFrame.new(), easingstyles.Sine, easingdirs.InOut, 0.2},
		},
		[4] = {
			["rarmweld"] = {CFrame.new(0.773609161, -0.926006079, -0.773324966, 0.192877501, 0.969730437, 0.149733886, 0.979573309, -0.181455955, -0.0866472423, -0.0568543673, 0.163387641, -0.984922051) * CFrame.new(0,0.3,0), easingstyles.Sine, easingdirs.In, 0.2},
			["larmweld"] = {CFrame.new(-0.780976295, -0.53411603, -0.816016912, 0.198409662, -0.971195996, -0.131948456, -0.976538301, -0.184390575, -0.111222014, 0.083688274, 0.150920749, -0.984996676) * CFrame.new(0,0.3,0), easingstyles.Sine, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.new(3.81469727e-06, 0.300005436, 2.38418579e-06, 0.999999285, 1.8812716e-07, -1.49011612e-07, 1.61118805e-07, 0.999998629, 8.19563866e-08, -1.1920929e-07, 8.94069672e-08, 0.999999583), easingstyles.Sine, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.new(), easingstyles.Sine, easingdirs.InOut, 0.2},
		},
		[5] = {
			["rarmweld"] = {CFrame.new(0.808786392, -1.05216312, -0.756877184, 0.358339041, 0.921504617, 0.149733901, 0.933181345, -0.348800063, -0.08664722, -0.0276186354, 0.170777932, -0.984921873), easingstyles.Elastic, easingdirs.Out, 0.5},
			["larmweld"] = {CFrame.new(-0.658809662, -0.689052343, -0.814886093, 0.518612206, -0.844765067, -0.131948262, -0.854579866, -0.507265925, -0.111221991, 0.0270232242, 0.170442179, -0.984996438), easingstyles.Elastic, easingdirs.Out, 0.5},
			["headtorootweld"] = {CFrame.new(3.81469727e-06, 0.300005436, 2.38418579e-06, 0.999999285, 1.8812716e-07, -1.49011612e-07, 1.61118805e-07, 0.999998629, 8.19563866e-08, -1.1920929e-07, 8.94069672e-08, 0.999999583), easingstyles.Sine, easingdirs.In, 0.15},
			["rootweld"] = {CFrame.new(), easingstyles.Sine, easingdirs.InOut, 0.2},
		},
	},
	["idle"] = {
		[1] = {
			["rarmweld"] = {CFrame.new(0.05,-0.25,0) * CFrame.new(1.14219666, -0.656560421, -0.909226418, 0.941444933, 0.325054795, -0.089532733, 0.258639753, -0.866617441, -0.426701248, -0.216293022, 0.378559977, -0.899948895) * CFrame.Angles(0,0,-0.2), easingstyles.Sine, easingdirs.Out, 0.3},
			["larmweld"] = {CFrame.new(-0.05,-0.25,0) * CFrame.new(-1.13857365, -0.698836803, -0.871101141, 0.9392398, -0.32580018, 0.108070657, -0.246695966, -0.859613478, -0.447438568, 0.238674939, 0.393592596, -0.887760043) * CFrame.Angles(0,0,0.2), easingstyles.Sine, easingdirs.Out, 0.3},
			["headtorootweld"] = {CFrame.new(7.62939453e-06, 0.300011635, 4.05311584e-06, 0.999998033, 2.83122063e-07, -2.01165676e-07, 2.86847353e-07, 0.999997616, 2.98023224e-07, -1.2665987e-07, 3.27825546e-07, 0.999999344), easingstyles.Sine, easingdirs.Out, 0.3},
			["rootweld"] = {CFrame.new(), easingstyles.Sine, easingdirs.Out, 0.35},
		}
	},
	["swing1"] = {
		[1] = {
			["rarmweld"] = {CFrame.new(1.8193531, -0.956459522, 0.145459175, 0.817060828, -0.467778027, -0.337039053, 0.261335403, 0.821556151, -0.506704092, 0.513921559, 0.325927824, 0.79350841), easingstyles.Sine, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-0.500593185, -0.755534649, -0.982423782, -0.000317309052, -0.937520564, -0.347929597, -0.998945475, -0.015676409, 0.0431522131, -0.0459104031, 0.347576439, -0.936526954), easingstyles.Sine, easingdirs.InOut, 0.2},
			["headtorootweld"] = {CFrame.new(9.53674316e-05, 0.299683571, 0.0137729645, 0.336097151, 0.0003172867, 0.941827297, 0.0431331694, 0.998945475, -0.015728889, -0.940838993, 0.0459104478, 0.335729033), easingstyles.Sine, easingdirs.InOut, 0.2},
			["rootweld"] = {CFrame.Angles(0,math.pi,0) * CFrame.new(0, 0, 0, 0.342020094, 0.163175911, -0.925416529, 1.21509947e-15, 0.98480773, 0.173648179, 0.939692616, -0.059391167, 0.33682403), easingstyles.Sine, easingdirs.Out, 0.2},
		},
		[2] = {
			["rarmweld"] = {CFrame.new(0.3,0,0) * CFrame.new(1.8193531, -0.956459522, 0.145459175, 0.817060828, -0.467778027, -0.337039053, 0.261335403, 0.821556151, -0.506704092, 0.513921559, 0.325927824, 0.79350841) * CFrame.new(0,-0.1,0.25) * CFrame.Angles(-0.3,0,0.4) * CFrame.new(0,-0.5,0), easingstyles.Sine, easingdirs.In, 0.2},
			["larmweld"] = {CFrame.new(-0.500593185, -0.755534649, -0.982423782, -0.000317309052, -0.937520564, -0.347929597, -0.998945475, -0.015676409, 0.0431522131, -0.0459104031, 0.347576439, -0.936526954), easingstyles.Sine, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.Angles(-0.2,-1.25,0) * CFrame.new(9.53674316e-05, 0.299683571, 0.0137729645, 0.336097151, 0.0003172867, 0.941827297, 0.0431331694, 0.998945475, -0.015728889, -0.940838993, 0.0459104478, 0.335729033), easingstyles.Sine, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.Angles(0.2,1.25,0) * CFrame.new(0, 0, 0, 0.342020094, 0.163175911, -0.925416529, 1.21509947e-15, 0.98480773, 0.173648179, 0.939692616, -0.059391167, 0.33682403) * CFrame.Angles(0,0,0.3), easingstyles.Sine, easingdirs.In, 0.2},
		},
		[3] = {
			["rarmweld"] = {CFrame.new(0.8,0.5,-0.6) * CFrame.new(2.21063137, -0.775531769, -0.171337128, 0.367661029, -0.924442887, -0.101145357, -0.324656218, -0.0256720409, -0.945483565, 0.871448994, 0.380455047, -0.30956465), easingstyles.Linear, easingdirs.In, 0.2},
			["larmweld"] = {CFrame.new(-0.457791328, -0.852360249, -1.09549522, -0.100215964, -0.923222423, -0.370967656, -0.978997767, 0.158026755, -0.128805578, 0.17753908, 0.350268334, -0.91966939), easingstyles.Linear, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.new(9.6321106e-05, 0.299683094, 0.0137748718, 0.770076036, 0.000317290425, -0.63795191, -0.0295327604, 0.998945475, -0.0351523384, 0.637267888, 0.0459104218, 0.76927346), easingstyles.Linear, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.Angles(0,-0.8,0) * CFrame.new(0, 0, 0, 0.75262934, 0.163175911, 0.637904823, -0.163175911, 0.98480773, -0.0593911707, -0.637904882, -0.059391167, 0.76782161):Inverse(), easingstyles.Linear, easingdirs.In, 0.2},
		},
		[4] = {
			["rarmweld"] = {CFrame.new(0,0,-0.3) * CFrame.new(1.02066994, -0.391653061, -0.954989433, 0.74294436, 0.661666453, -0.101145402, 0.135162801, -0.296296686, -0.945483506, -0.655563653, 0.688771009, -0.30956459) * CFrame.new(0,0.5,0) * CFrame.Angles(0.35,0,0) * CFrame.new(0,-0.5,0), easingstyles.Sine, easingdirs.Out, 0.5},
			["larmweld"] = {CFrame.new(-0.457794189, -0.852359772, -1.09549522, -0.100215927, -0.923222184, -0.370967686, -0.978997767, 0.158026785, -0.128805608, 0.177539051, 0.350268066, -0.919669509), easingstyles.Sine, easingdirs.Out, 0.3},
			["headtorootweld"] = {CFrame.new(9.3460083e-05, 0.299683571, 0.0137748718, 0.179844856, 0.000317305326, -0.983694673, -0.0452188775, 0.998945534, -0.00794497877, 0.982654929, 0.0459104329, 0.17966935), easingstyles.Sine, easingdirs.Out, 0.5},
			["rootweld"] = {CFrame.new(0, 0, 0, -0.497520864, 0.163175911, 0.851966143, 0.0301536936, 0.98480773, -0.171010062, -0.866927624, -0.059391167, -0.494882882):Inverse(), easingstyles.Sine, easingdirs.Out, 0.4},
		},
		[5] = {
			["rootweld"] = {CFrame.Angles(0,0.3,0) * CFrame.new(0, 0, 0, -0.405218601, 0.103055239, 0.908392668, -0.0173312314, 0.992581785, -0.120337442, -0.914055407, -0.0645065457, -0.400426596):Inverse(), easingstyles.Sine, easingdirs.In, 0.4},
		}
	},
	["swing2"] = {
		[1] = {
			["rarmweld"] = {CFrame.new(0.258669853, -0.760544062, -0.878656387, 0.000317294151, 0.985881448, 0.167444438, 0.998945236, -0.00799995661, 0.0452092141, 0.0459103882, 0.167253494, -0.984844267), easingstyles.Sine, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-1.87974262, -0.840487242, -0.0787315369, 0.866379499, 0.498808712, -0.0240071714, -0.4916583, 0.843561947, -0.21604313, -0.0875126123, 0.198978633, 0.976088524), easingstyles.Sine, easingdirs.InOut, 0.2},
			["headtorootweld"] = {CFrame.new(9.53674316e-05, 0.299684048, 0.0137729645, 0.17984499, 0.000317282975, -0.983694911, -0.0452189147, 0.998945236, -0.00794496946, 0.982654989, 0.0459104031, 0.179669634), easingstyles.Sine, easingdirs.InOut, 0.2},
			["rootweld"] = {CFrame.Angles(0,-0.5,0) * CFrame.new(0, 0, 0, 0.173648223, -0.171010047, 0.969846249, 6.16922418e-16, 0.984807611, 0.173648134, -0.98480773, -0.0301536862, 0.171010092):Inverse(), easingstyles.Sine, easingdirs.Out, 0.2},
		},
		[2] = {
			["rarmweld"] = {CFrame.new(0.293008804, -0.644740582, -1.07552338, -0.0569713414, 0.985881388, 0.157454789, 0.923239052, -0.00800001621, 0.38414225, 0.379978448, 0.167253464, -0.909748435), easingstyles.Sine, easingdirs.In, 0.2},
			["larmweld"] = {CFrame.new(-2.70503426, -1.54852104, -0.255741119, 0.500902295, 0.865170598, -0.0240071919, -0.847569466, 0.484716713, -0.216043085, -0.175277486, 0.128564209, 0.976088464), easingstyles.Sine, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.new(9.72747803e-05, 0.299683571, 0.0137729645, 0.985881388, 0.000317215919, -0.167444438, -0.00800001621, 0.998945236, -0.0452091508, 0.16725345, 0.0459104329, 0.984844208), easingstyles.Sine, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.new(0, 0, 0, 0.940601826, -0.339501619, -0.00259810686, 0.339422047, 0.940150559, 0.0301536769, -0.0077946363, -0.0292444509, 0.999541819):Inverse(), easingstyles.Sine, easingdirs.In, 0.2},
		},
		[3] = {
			["rarmweld"] = {CFrame.new(0,0,-0.5) * CFrame.new(0.293007851, -0.644741058, -1.07552338, -0.0569712073, 0.985881269, 0.157454744, 0.923238873, -0.00799991935, 0.38414216, 0.379978538, 0.167253539, -0.909748495), easingstyles.Linear, easingdirs.In, 0.2},
			["larmweld"] = {CFrame.new(-2.41886711, -0.342449903, -1.49530792, 0.0644817352, 0.931209743, -0.358733565, -0.926526666, -0.0776570663, -0.368125677, -0.370660722, 0.35611397, 0.857783854), easingstyles.Linear, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.new(9.53674316e-05, 0.299683571, 0.0137748718, 0.167444333, 0.000317298574, 0.985881269, 0.04520908, 0.998945057, -0.00799990445, -0.984844327, 0.0459104776, 0.167253479), easingstyles.Linear, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.new(0, 0, 0, -0.00259814807, -0.33950159, -0.940601826, 0.030153662, 0.940150499, -0.339422047, 0.999541819, -0.029244449, 0.00779459253):Inverse(), easingstyles.Linear, easingdirs.In, 0.2},
		},
		[4] = {
			["rarmweld"] = {CFrame.new(0.293007851, -0.644740582, -1.07552147, -0.0569711924, 0.985881329, 0.157454699, 0.923238873, -0.00799992681, 0.38414225, 0.379978448, 0.167253584, -0.909748495), easingstyles.Linear, easingdirs.Out, 0.2},
			["larmweld"] = {CFrame.new(-1.54858112, -0.248359442, -1.57910156, 0.0644817352, -0.018606782, -0.997745037, -0.926526725, -0.37248525, -0.0529326014, -0.370660663, 0.927851319, -0.0412583947), easingstyles.Linear, easingdirs.Out, 0.4},
			["headtorootweld"] = {CFrame.new(0.0146312714, 0.296020985, -0.0720825195, 0.164845377, 0.0293888878, 0.985881329, -0.128942654, 0.991619229, -0.0079999119, -0.97785449, -0.125803471, 0.167253524), easingstyles.Linear, easingdirs.Out, 0.5},
			["rootweld"] = {CFrame.Angles(0,0.5,0) * CFrame.new(0, 0, 0, -0.49488315, -0.171010226, -0.851966023, -0.059391316, 0.984807491, -0.16317603, 0.866927505, -0.0301536769, -0.497521222):Inverse(), easingstyles.Sine, easingdirs.Out, 0.4},
		},
		[5] = {
			["rarmweld"] = {CFrame.new(0.293007851, -0.644740105, -1.07552528, -0.0569712222, 0.985881329, 0.157454789, 0.923238754, -0.00799993612, 0.38414222, 0.379978418, 0.167253509, -0.909748435), easingstyles.Sine, easingdirs.Out, 0.3},
			["larmweld"] = {CFrame.new(-0.18221283, -0.101358414, -1.31591225, 0.215162277, -0.848910213, -0.482758462, -0.872149765, -0.389447361, 0.296115398, -0.439384729, 0.357325077, -0.824171841) * CFrame.new(0,0.25,0.25) * CFrame.Angles(-0.35,0,0) * CFrame.new(0,-0.5,0), easingstyles.Sine, easingdirs.Out, 0.5},
			["headtorootweld"] = {CFrame.new(9.53674316e-05, 0.299684048, 0.0137729645, 0.167444378, 0.000317289494, 0.985881269, 0.0452091843, 0.998944998, -0.00799992308, -0.984844089, 0.0459104143, 0.16725345), easingstyles.Sine, easingdirs.Out, 0.5},
			["rootweld"] = {CFrame.new(0, 0, 0, -0.168735921, -0.0675501823, -0.98334378, -0.0408176556, 0.997271717, -0.0615029037, 0.984815717, 0.0297600403, -0.171032846):Inverse(), easingstyles.Sine, easingdirs.In, 0.5},
		}
	},
	["swing3"] = {
		[1] = {
			["rarmweld"] = {CFrame.new(0,0.5,0) * CFrame.new(1.71041489, -0.37266469, -0.396123886, -0.0343486965, 0.653690755, 0.755981803, 0.858640194, -0.367783576, 0.357032269, 0.51142627, 0.661379933, -0.548652232), easingstyles.Sine, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-2.09034348, -0.793517351, -0.144979477, -0.000317277387, 0.985881388, -0.167444542, -0.998945534, -0.00799992401, -0.0452091917, -0.0459104031, 0.167253584, 0.984844208), easingstyles.Sine, easingdirs.InOut, 0.2},
			["headtorootweld"] = {CFrame.new(9.53674316e-05, 0.299683571, 0.0137748718, 0.336097151, 0.000317288563, 0.941827238, 0.0431331918, 0.998945534, -0.0157288928, -0.940838933, 0.0459104031, 0.335729003), easingstyles.Sine, easingdirs.InOut, 0.2},
			["rootweld"] = {CFrame.Angles(0,0.55,0) * CFrame.new(0, 0, 0, 0.173648223, -0.171010062, -0.969846249, 6.16922418e-16, 0.98480773, -0.173648179, 0.98480773, 0.0301536974, 0.171010107):Inverse(), easingstyles.Sine, easingdirs.Out, 0.2},
		},
		[2] = {
			["rarmweld"] = {CFrame.new(0,0,-1) * CFrame.new(1.28547096, -0.617397785, -1.94119453, 0.679072976, -0.00301932544, 0.734064579, 0.725914836, 0.151412472, -0.670910835, -0.109120853, 0.988465667, 0.105011925), easingstyles.Sine, easingdirs.In, 0.2},
			["larmweld"] = {CFrame.new(-0.844640732, -0.752892494, -1.03752518, -0.000317335129, -0.761982381, -0.647597551, -0.998945415, -0.0294899642, 0.0351881832, -0.0459104329, 0.646925807, -0.761169493), easingstyles.Sine, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.Angles(0,-0.4,0) * CFrame.new(9.53674316e-05, 0.299683571, 0.0137748718, 0.336097151, 0.000317288563, 0.941827238, 0.0431331918, 0.998945534, -0.0157288928, -0.940838933, 0.0459104031, 0.335729003), easingstyles.Sine, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.new(0, 0, 0, 0.940601826, -0.339501619, -0.00259810686, 0.339422047, 0.940150559, 0.0301536769, -0.0077946363, -0.0292444509, 0.999541819):Inverse(), easingstyles.Sine, easingdirs.In, 0.2},
		},
		[3] = {
			["rarmweld"] = {CFrame.new(0,-0.25,0) * CFrame.new(1.46557045, -1.3237462, -1.85366821, 0.679072917, -0.253901958, 0.688762307, 0.725914836, 0.371746182, -0.578663886, -0.109120831, 0.892937958, 0.436753988) * CFrame.new(0,-0.5,0) * CFrame.Angles(-0.5,0,0.3), easingstyles.Linear, easingdirs.In, 0.2},
			["larmweld"] = {CFrame.new(-0.38824749, -0.388237953, -1.10569, -0.174239367, -0.975572824, -0.133782715, -0.84353596, 0.0777836889, 0.531410098, -0.508023441, 0.205443129, -0.836483836), easingstyles.Linear, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.new(9.44137573e-05, 0.299683571, 0.0137710571, 0.347929716, 0.000317291706, -0.937520444, -0.0431522429, 0.998945355, -0.0156764537, 0.936526954, 0.0459105074, 0.347576559), easingstyles.Linear, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.Angles(0,-0.2,0) * CFrame.new(0, 0, 0, -0.00259809569, -0.339501619, 0.940601826, 0.030153688, 0.940150678, 0.339422107, -0.999541819, 0.0292444639, 0.00779463537):Inverse(), easingstyles.Linear, easingdirs.In, 0.2},
		},
		[4] = {
			["rarmweld"] = {CFrame.new(0.559087753, -1.28297615, -1.33488655, -0.163016468, 0.787117541, 0.594870746, 0.84206152, 0.425204247, -0.331863284, -0.514156878, 0.44681868, -0.732116818) * CFrame.new(0,0.25,0.25) * CFrame.Angles(-0.1,0,0) * CFrame.new(0,-0.5,0), easingstyles.Linear, easingdirs.Out, 0.4},
			["larmweld"] = {CFrame.new(-0.121709824, -0.656519175, -0.9893713, -0.00445670635, -0.978128791, 0.20795095, -0.874974251, 0.104495563, 0.472758532, -0.484148949, -0.179844752, -0.856303275), easingstyles.Linear, easingdirs.Out, 0.4},
			["headtorootweld"] = {CFrame.new(9.3460083e-05, 0.299683809, 0.0137729645, 0.00629606843, 0.000317245722, -0.999979913, -0.045911476, 0.998945355, 2.79098749e-05, 0.998925567, 0.0459105074, 0.00630405545), easingstyles.Linear, easingdirs.Out, 0.5},
			["rootweld"] = {CFrame.Angles(0,-0.6,0) * CFrame.new(0, 0, 0, -0.00259809569, -0.339501619, 0.940601826, 0.030153688, 0.940150678, 0.339422107, -0.999541819, 0.0292444639, 0.00779463537):Inverse(), easingstyles.Sine, easingdirs.Out, 0.5},
		},
		[5] = {
			["rarmweld"] = {CFrame.new(-0.5,0,0) * CFrame.new(0.365228653, -1.17369819, -1.5460577, 0.0211739242, 0.950483143, 0.310053468, 0.705356836, 0.205580592, -0.678386211, -0.708535552, 0.233062446, -0.666076899) * CFrame.new(0,0.4,0) * CFrame.Angles(0.25,0,0) * CFrame.new(0,-0.5,0), easingstyles.Sine, easingdirs.Out, 0.5},
			["larmweld"] = {CFrame.new(-0.121709824, -0.656519175, -0.9893713, -0.00445670635, -0.978128791, 0.20795095, -0.874974251, 0.104495563, 0.472758532, -0.484148949, -0.179844752, -0.856303275), easingstyles.Linear, easingdirs.Out, 0.4},
			["headtorootweld"] = {CFrame.new(9.3460083e-05, 0.299683809, 0.0137729645, 0.00629606843, 0.000317245722, -0.999979913, -0.045911476, 0.998945355, 2.79098749e-05, 0.998925567, 0.0459105074, 0.00630405545), easingstyles.Sine, easingdirs.Out, 0.5},
			["rootweld"] = {CFrame.new(0, 0, 0, -0.472550929, -0.19311817, 0.859884202, -0.143597186, 0.979529321, 0.141074777, -0.869525909, -0.0568119511, -0.490608722):Inverse(), easingstyles.Sine, easingdirs.In, 0.5},
		}
	},
	["swing4"] = {
		[1] = {
			["rarmweld"] = {CFrame.new(1.96870899, -0.686678886, -0.484977722, -0.0601048991, -0.937520623, 0.342698783, 0.991262615, -0.015676491, 0.130968392, -0.11741326, 0.347576201, 0.930271149), easingstyles.Sine, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-0.25,0.5,0) * CFrame.new(-1.81323242, -0.394709587, -0.505847931, 0.221193433, -0.761982322, -0.608651161, -0.950736701, -0.0294900239, -0.308593571, 0.217193738, 0.646925867, -0.730967581) * CFrame.new(0,-0.3,0) * CFrame.Angles(0,0,-0.2), easingstyles.Sine, easingdirs.InOut, 0.2},
			["headtorootweld"] = {CFrame.new(9.53674316e-05, 0.299683571, 0.0137748718, 0.179844946, 0.0003172867, -0.983694851, -0.04521887, 0.998945534, -0.00794497132, 0.982654989, 0.045910418, 0.179669648), easingstyles.Sine, easingdirs.InOut, 0.2},
			["rootweld"] = {CFrame.Angles(0,-0.3,0) * CFrame.new(0, 0, 0, -0.173647955, 0.171010077, 0.969846308, -6.16921465e-16, 0.98480773, -0.173648179, -0.984807789, -0.0301536508, -0.171009853):Inverse(), easingstyles.Sine, easingdirs.Out, 0.2},
		},
		[2] = {
			["rarmweld"] = {CFrame.new(1.494627, -0.663722038, -0.723251343, -0.058952041, 0.011796671, 0.998191059, 0.996642888, -0.0562108345, 0.0595249012, 0.0568113551, 0.99834919, -0.00844331924), easingstyles.Sine, easingdirs.In, 0.2},
			["larmweld"] = {CFrame.new(0,1,-0.8) * CFrame.new(-1.12195206, -0.860088825, -2.28838062, 0.221920475, 0.015228577, -0.974945664, -0.974051058, 0.049043458, -0.220950782, 0.0444499888, 0.998680532, 0.025717156) * CFrame.Angles(0,0,-0.2), easingstyles.Sine, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.new(9.53674316e-05, 0.299683571, 0.0137729645, 0.999980092, 0.000317290425, 0.00629592501, -2.78949738e-05, 0.998945534, -0.0459115282, -0.00630385475, 0.0459104404, 0.998925745), easingstyles.Sine, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.Angles(0.15,0,0) * CFrame.new(0, 0, 0, 0.985265851, 0.171010077, 0.00259785354, -0.171010077, 0.98480773, 0.0301536825, 0.00259819627, -0.0301536508, 0.999541938):Inverse(), easingstyles.Sine, easingdirs.In, 0.2},
		},
		[3] = {
			["rarmweld"] = {CFrame.new(0.175883293, -0.739398956, -0.897779465, -0.0861382484, 0.985074878, 0.149023026, 0.970094025, 0.0488596633, 0.23776041, 0.226930603, 0.165046617, -0.959823728), easingstyles.Linear, easingdirs.In, 0.2},
			["larmweld"] = {CFrame.new(0,-0.25,-0.4) * CFrame.new(-0.954066277, -1.563344, -1.96582794, 0.199803069, -0.0977719128, -0.974945724, -0.819031, 0.529498219, -0.220950738, 0.537835002, 0.842657626, 0.0257171579), easingstyles.Linear, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.new(9.44137573e-05, 0.299684525, 0.0137729645, 0.336097151, 0.0003172906, 0.941827238, 0.0431331694, 0.998945296, -0.0157288834, -0.940839052, 0.0459104478, 0.335729033), easingstyles.Linear, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.Angles(0.15,0.2,0) * CFrame.new(0, 0, 0, 0.00259780884, 0.171010062, -0.985265851, 0.030153688, 0.98480767, 0.171010077, 0.999541938, -0.0301536489, -0.00259824097):Inverse(), easingstyles.Linear, easingdirs.In, 0.2},
		},
		[4] = {
			["rarmweld"] = {CFrame.new(0.175883293, -0.739398956, -0.897781372, -0.0861382633, 0.985074759, 0.149023116, 0.970093966, 0.0488596708, 0.237760425, 0.226930633, 0.165046513, -0.959823847), easingstyles.Linear, easingdirs.Out, 0.4},
			["larmweld"] = {CFrame.new(-0.289923668, -1.16539955, -1.60655212, 0.19980301, -0.949589372, -0.241575509, -0.819031, -0.0265266635, -0.573135257, 0.537835062, 0.312372237, -0.783043444) * CFrame.new(0,0.5,0) * CFrame.Angles(-0.3,0,0) * CFrame.new(0,-0.5,0), easingstyles.Linear, easingdirs.Out, 0.4},
			["headtorootweld"] = {CFrame.new(9.3460083e-05, 0.299683571, 0.0137729645, 0.336097211, 0.000317305326, 0.941827297, 0.0431331694, 0.998945355, -0.0157288909, -0.940839171, 0.0459104478, 0.335728973), easingstyles.Linear, easingdirs.Out, 0.5},
			["rootweld"] = {CFrame.Angles(0.1,0.2,0) * CFrame.new(0, 0, 0, -0.490383327, 0.171010062, -0.854564071, 0.111618929, 0.98480767, 0.133022219, 0.864329517, -0.0301536489, -0.502021313):Inverse(), easingstyles.Sine, easingdirs.Out, 0.5},
		},
		[5] = {
			["rarmweld"] = {CFrame.new(0.175884247, -0.73939991, -0.897777557, -0.0861382782, 0.985074937, 0.149023056, 0.970094025, 0.0488596708, 0.237760425, 0.226930618, 0.165046558, -0.959823787), easingstyles.Sine, easingdirs.Out, 0.5},
			["larmweld"] = {CFrame.new(-0.0661373138, -0.978745461, -1.4054451, 0.199803084, -0.977112234, -0.0730109736, -0.819031, -0.12564756, -0.559821725, 0.537835062, 0.171652526, -0.825390041), easingstyles.Linear, easingdirs.Out, 0.4},
			["headtorootweld"] = {CFrame.new(9.3460083e-05, 0.299683571, 0.0137729645, 0.336097211, 0.000317305326, 0.941827297, 0.0431331694, 0.998945355, -0.0157288909, -0.940839171, 0.0459104478, 0.335728973), easingstyles.Sine, easingdirs.Out, 0.5},
			["rootweld"] = {CFrame.Angles(0,0.2,0) * CFrame.new(0, 0, 0, -0.147236675, 0.171010062, -0.974205792, 0.0558042526, 0.98480767, 0.16443716, 0.98752594, -0.0301536489, -0.154542953):Inverse(), easingstyles.Sine, easingdirs.In, 0.5},
		}
	},
	["block"] = {
		[1] = {
			["rarmweld"] = {CFrame.new(0.592468262, -0.138791561, -1.10939217, 0.0805527121, 0.086837925, -0.992960513, 0.0533224009, -0.995146632, -0.0827033892, -0.995323002, -0.0462850519, -0.0847921595), easingstyles.Quad, easingdirs.Out, 0.15},
			["larmweld"] = {CFrame.new(-0.606412888, -0.138757706, -1.1018343, 0.0930918008, -0.0874700919, 0.991807997, -0.05332743, -0.995141685, -0.082758747, 0.994228423, -0.0451864153, -0.0973040909), easingstyles.Quad, easingdirs.InOut, 0.15},
			["headtorootweld"] = {CFrame.new(-0.00018119812, 0.299783707, -0.0298452377, 0.999980152, -0.000232636245, 0.00629954739, -2.79046071e-05, 0.999145627, 0.0413270146, -0.00630377978, -0.0413263701, 0.999125838), easingstyles.Quad, easingdirs.InOut, 0.15},
			["rootweld"] = {CFrame.new(0, 0, 0, 1, 3.5391944e-15, -3.09639352e-16, 3.55271368e-15, 0.996194661, -0.0871557295, 0, 0.0871557295, 0.996194661):Inverse(), easingstyles.Quad, easingdirs.Out, 0.15},
		},
		[2] = {
			["rarmweld"] = {CFrame.new(0,0,0.35) *CFrame.new(0.577981949, -0.0649819374, -1.01174641, 0.0642498881, 0.0995064899, -0.992960572, 0.225317746, -0.97076875, -0.0827033445, -0.972164452, -0.218417883, -0.0847923756), easingstyles.Quad, easingdirs.Out, 0.15},
			["larmweld"] = {CFrame.new(0,0,0.35) * CFrame.new(-0.590696335, -0.0649495125, -1.00437927, 0.0764884949, -0.10230644, 0.991807997, -0.225321829, -0.970763087, -0.0827587545, 0.971277237, -0.217145875, -0.097304076), easingstyles.Quad, easingdirs.InOut, 0.15},
			["headtorootweld"] = {CFrame.Angles(0.25,0,0) * CFrame.new(-0.000453948975, 0.29608202, -0.0733060837, 0.999980152, -0.000780792616, 0.00625530025, -2.79046071e-05, 0.991741717, 0.128251016, -0.00630377978, -0.128248647, 0.991721988), easingstyles.Quad, easingdirs.InOut, 0.15},
			["rootweld"] = {CFrame.Angles(-0.15,0,0) * CFrame.new(0, 0, 0, 1, 3.4987401e-15, -6.16920671e-16, 3.55271368e-15, 0.984807789, -0.173647732, 0, 0.173647732, 0.984807789):Inverse(), easingstyles.Back, easingdirs.Out, 0.2},
		},
	},
	["kick"] = { --kumitsu w wykonaniu marka
		[1] = {
			["rlegweld"] = {CFrame.new(1.07978344, -1.82250786, 0.18944931, 0.772082388, -0.63282609, -0.0584788322, 0.625167251, 0.739736915, 0.248908296, -0.114256769, -0.228736803, 0.96675998), easingstyles.Sine, easingdirs.In, 0.2},
			["llegweld"] = {CFrame.new(-0.505302429, -0.2, -0.833242416, 0.999980152, 0.00140567869, 0.00614514947, -2.79257074e-05, 0.975796878, -0.218678921, -0.00630378723, 0.218674332, 0.975777507), easingstyles.Sine, easingdirs.InOut, 0.2},
			["rarmweld"] = {CFrame.new(1.65428734, -0.920772552, -0.298259735, 0.96291858, -0.264468461, -0.0533301197, 0.145290941, 0.67488724, -0.723476052, 0.227328509, 0.688900292, 0.688286543) * CFrame.new(0,0.5,0) * CFrame.Angles(0.2,0,-0.8) * CFrame.new(0,-0.5,0), easingstyles.Linear, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-1.76321411, -1.01451969, 0.659912109, 0.920055866, 0.388565361, 0.050142318, -0.319165409, 0.669124007, 0.671123326, 0.227223814, -0.633474588, 0.739648104) * CFrame.new(0,0.5,0) * CFrame.Angles(-0.4,0,0) * CFrame.new(0,-0.5,0), easingstyles.Sine, easingdirs.In, 0.2},
			["headtorootweld"] = {CFrame.new(0.00366592407, 0.221743584, -0.29605484, 0.568533659, -0.154157385, 0.80808717, -0.466278493, 0.74888134, 0.470915318, -0.677756548, -0.644524992, 0.353884012), easingstyles.Sine, easingdirs.In, 0.2},
			["rootweld"] = {CFrame.Angles(0,1.3,0) *CFrame.new(0, 0, 0, 0.342020154, 1.60882965e-14, -0.939692616, -0.604022384, 0.766044796, -0.219846189, 0.719846606, 0.642787218, 0.262002736):Inverse(), easingstyles.Linear, easingdirs.InOut, 0.2},
		},
		[2] = {
			["rlegweld"] = {CFrame.new(1.26793194, -1.7824192, 0.0938949585, 0.649996817, -0.759930313, -0.00310220569, 0.747401774, 0.638531387, 0.183488995, -0.137458026, -0.121585816, 0.983016789), easingstyles.Linear, easingdirs.InOut, 0.2},
			["llegweld"] = {CFrame.new(-1.77346611, -1.26206398, -1.20810509, 0.498772621, 0.854610682, -0.144451037, -0.845078707, 0.442489117, -0.300075114, -0.192529425, 0.271741807, 0.942915022) * CFrame.new(0,-0.25,0), easingstyles.Sine, easingdirs.In, 0.2},
			["rarmweld"] = {CFrame.new(1.65428734, -0.920772552, -0.298259735, 0.96291858, -0.264468461, -0.0533301197, 0.145290941, 0.67488724, -0.723476052, 0.227328509, 0.688900292, 0.688286543) * CFrame.new(0,0.5,0) * CFrame.Angles(0.2,0,-0.8) * CFrame.new(0,-0.5,0), easingstyles.Linear, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-1.72185946, -1.09910393, 0.373657227, 0.920055747, 0.391369283, -0.0180930533, -0.319165468, 0.775497675, 0.544735372, 0.227223843, -0.49541226, 0.838412702), easingstyles.Linear, easingdirs.InOut, 0.2},
			["headtorootweld"] = {CFrame.new(0.0802221298, 0.278718948, -0.546686172, 0.654862463, -0.0530905165, 0.753880799, -0.545951307, 0.656535685, 0.520478725, -0.522582293, -0.752424419, 0.400955796), easingstyles.Linear, easingdirs.InOut, 0.2},
			["rootweld"] = {CFrame.Angles(0,0.4,0) * CFrame.new(-0.0151634216, 0.139906645, -0.0140075684, 0.0587833077, -0.102975652, -0.992945373, -0.75642705, 0.644483745, -0.111618839, 0.651431143, 0.757652104, -0.0400086865):Inverse(), easingstyles.Linear, easingdirs.In, 0.2},
		},
		[3] = {
			["rlegweld"] = {CFrame.new(1.24500084, -1.86332321, 0.207344055, 0.709668875, -0.69942683, -0.0846872926, 0.697731733, 0.681055486, 0.22211197, -0.0976743698, -0.216714978, 0.971336305), easingstyles.Linear, easingdirs.InOut, 0.25},
			["llegweld"] = {CFrame.new(-1.5,0,0) * CFrame.new(-2.38225842, -1.82050228, -0.245594025, 0.499743909, 0.863948405, 0.062039867, -0.748895347, 0.466958523, -0.47021848, -0.435214579, 0.18852748, 0.880366743), easingstyles.Linear, easingdirs.In, 0.25},
			["rarmweld"] = {CFrame.new(1.65428734, -0.920772552, -0.298259735, 0.96291858, -0.264468461, -0.0533301197, 0.145290941, 0.67488724, -0.723476052, 0.227328509, 0.688900292, 0.688286543) * CFrame.new(0,0.5,0) * CFrame.Angles(0.2,0,-0.8) * CFrame.new(0,-0.5,0), easingstyles.Linear, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-1.97337246, -1.05799294, -0.33102417, 0.617001951, 0.767584562, -0.173558682, -0.781746566, 0.623169363, -0.0230704732, 0.0904478878, 0.149913386, 0.984553218), easingstyles.Linear, easingdirs.InOut, 0.25},
			["headtorootweld"] = {CFrame.new(0.0802221298, 0.278717995, -0.54668808, 0.654862523, -0.053090483, 0.753880858, -0.545951486, 0.656535625, 0.520478725, -0.522582233, -0.752424359, 0.400955826), easingstyles.Linear, easingdirs.InOut, 0.25},
			["rootweld"] = {CFrame.Angles(0,-0.2,0.05) * CFrame.new(-0.0151634216, 0.139906883, -0.0140066147, 0.230313376, -0.102975652, -0.967652678, -0.725552857, 0.644483745, -0.241275251, 0.648481905, 0.757652104, 0.0737189576):Inverse(), easingstyles.Linear, easingdirs.Out, 0.25},
		},
		[4] = {
			["rlegweld"] = {CFrame.new(1.07329369, -1.88901329, 0.0674715042, 0.852267504, -0.513790488, -0.0982815027, 0.522190809, 0.846735001, 0.10176675, 0.0309315324, -0.138053998, 0.989941657), easingstyles.Linear, easingdirs.InOut, 0.3},
			["llegweld"] = {CFrame.new(0,0.25,0) * CFrame.new(-0.808603287, -1.91098785, 0.254718781, -0.97578454, 0.121206403, 0.182082146, 0.0205047429, 0.879454851, -0.475540698, -0.217771351, -0.460291684, -0.860643625) * CFrame.Angles(0,-0.3,0), easingstyles.Linear, easingdirs.In, 0.3},
			["rarmweld"] = {CFrame.new(1.65428734, -0.920772552, -0.298259735, 0.96291858, -0.264468461, -0.0533301197, 0.145290941, 0.67488724, -0.723476052, 0.227328509, 0.688900292, 0.688286543) * CFrame.new(0,0.5,0) * CFrame.Angles(0.2,0,-0.8) * CFrame.new(0,-0.5,0), easingstyles.Linear, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-1.52203751, -1.19352913, -0.733335495, 0.986365497, 0.0443205684, -0.15849036, -0.139427915, 0.736666799, -0.661726415, 0.087426275, 0.674801826, 0.732802033) * CFrame.new(0,-0.5,0) * CFrame.Angles(0.5,0,0) * CFrame.new(0,0.5,0), easingstyles.Linear, easingdirs.InOut, 0.3},
			["headtorootweld"] = {CFrame.new(-0.10990715, 0.358057022, -0.142168045, 0.871501625, -0.256148815, 0.418177903, 0.0720208287, 0.910349011, 0.407526433, -0.485075384, -0.325042337, 0.811818719), easingstyles.Linear, easingdirs.InOut, 0.3},
			["rootweld"] = {CFrame.new(-0.0151634216, 0.139906883, -0.0140066147, 0.741466045, 0.336808473, -0.580334604, -0.481542856, 0.869407535, -0.110666811, 0.467273653, 0.361511648, 0.80682379):Inverse(), easingstyles.Linear, easingdirs.Out, 0.3},
		}
	},
	["rlegkick"] = {
		[1] = {
			["rlegweld"] = {CFrame.new(-0.1,0.5,0.5) * CFrame.new(0.434185028, -1.44070339, -1.11082172, 0.995897233, 0.0660540164, -0.0618507117, -0.0879529193, 0.545815349, -0.83327657, -0.021282196, 0.835297763, 0.549385667) * CFrame.Angles(-1.5,0,0), easingstyles.Sine, easingdirs.InOut, 0.2},
			["llegweld"] = {CFrame.new(-0.560279846, -2.014503, -0.16561985, 0.998222828, 0.0593911596, -0.00488269329, -0.0593911633, 0.98480767, -0.163175792, -0.00488272309, 0.163175777, 0.986584842) * CFrame.new(0,1,0) * CFrame.Angles(-0.3,0,0) * CFrame.new(0,-1,0), easingstyles.Sine, easingdirs.InOut, 0.2},
			["rarmweld"] = {CFrame.new(0.884101868, -1.02058077, -1.05841112, 0.934229195, 0.355087101, -0.0335994661, 0.331458509, -0.899113894, -0.285883456, -0.131723285, 0.255943924, -0.957675219), easingstyles.Sine, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-0.897422791, -1.02053165, -1.04719448, 0.935797215, -0.35188207, 0.0215073228, -0.331510425, -0.899094284, -0.285885096, 0.119934946, 0.260400593, -0.958022535), easingstyles.Sine, easingdirs.InOut, 0.2},
			["headtorootweld"] = {CFrame.new(9.53674316e-05, 0.299683571, 0.0137729645, 0.937520564, 0.0003172867, 0.347929746, 0.0156764518, 0.998945475, -0.0431522839, -0.347576559, 0.0459104478, 0.936527073) * CFrame.new(0,-0.5,0) * CFrame.Angles(-0.3,0,0) * CFrame.new(0,0.5,0), easingstyles.Cubic, easingdirs.InOut, 0.2},
			["rootweld"] = {CFrame.Angles(-0.3,0.35,0) * CFrame.new(0, 0, 0, 0.939692616, 4.58613894e-14, -0.342020154, 0.0593911633, 0.98480773, 0.163175881, 0.336824089, -0.173648149, 0.925416589):Inverse(), easingstyles.Cubic, easingdirs.InOut, 0.2},
		},
		[2] = {
			["rlegweld"] = {CFrame.new(0.25,-0.5,-0.5) * CFrame.Angles(0.2,0,0.3) * CFrame.new(1.17900658, -0.405755043, -2.06898308, 0.945147216, -0.32218799, -0.0537684038, -0.0876033753, -0.0914451778, -0.991949201, 0.314677209, 0.942248583, -0.114653967) * CFrame.new(0,0,0.65) * CFrame.Angles(-0.35,0.15,0), easingstyles.Sine, easingdirs.InOut, 0.15},
			["llegweld"] = {CFrame.new(-0.453650475, -1.97300601, 0.0689945221, 0.998540282, -0.0470799655, -0.0264661312, 0.04850135, 0.997256994, 0.0559104383, 0.023761183, -0.0571124852, 0.998084843) * CFrame.new(0,1,0) * CFrame.Angles(-0.3,0,0) * CFrame.new(0,-1,0), easingstyles.Sine, easingdirs.In, 0.2},
			["rarmweld"] = {CFrame.new(1.34953117, -0.862673283, -0.591186523, 0.732581794, 0.478630841, 0.483979374, 0.555674016, -0.831180513, -0.0191082079, 0.393128514, 0.282933086, -0.874870658), easingstyles.Sine, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.Angles(-0.2,0,0) * CFrame.new(-1.14703178, -1.05201626, -0.715093613, 0.87479943, -0.479164958, -0.0715995729, -0.484180301, -0.859410226, -0.164266229, 0.0171771348, 0.178367198, -0.983813941), easingstyles.Sine, easingdirs.InOut, 0.2},
			["headtorootweld"] = {CFrame.new(9.53674316e-05, 0.299683571, 0.0137729645, 0.941827118, 0.000317279249, -0.336096942, -0.0157288834, 0.998945415, -0.043133188, 0.335728765, 0.0459104255, 0.940839052) * CFrame.new(0,-0.5,0) * CFrame.Angles(-0.4,0,0) * CFrame.new(0,0.5,0), easingstyles.Sine, easingdirs.InOut, 0.2},
			["rootweld"] = {CFrame.Angles(-0.45,-0.2,0) * CFrame.new(0, 0, 0, 0.939692438, 0.059391208, 0.336824268, -0.0593912005, 0.998181462, -0.0103132129, -0.336824298, -0.0103131533, 0.941510975):Inverse(), easingstyles.Sine, easingdirs.InOut, 0.2},
		},
		[3] = {
			["rlegweld"] = {CFrame.new(-1.5,0.4,0.25) * CFrame.new(0.880243301, -1.1351223, -1.34087372, 0.985295773, -0.166134238, -0.0398876481, 0.0871415064, 0.689455032, -0.719067454, 0.146962419, 0.705018342, 0.693794489) * CFrame.new(0.5,-0.5,0) * CFrame.Angles(0,0,0.2), easingstyles.Linear, easingdirs.InOut, 0.4},
			["llegweld"] = {CFrame.new(-0.453649521, -1.97300625, 0.0689926147, 0.998540223, -0.0470799692, -0.0264661014, 0.0485013463, 0.997256994, 0.0559104383, 0.023761183, -0.0571124814, 0.998084843) * CFrame.new(0,1,0) * CFrame.Angles(-0.3,0,0) * CFrame.new(0,-1,0), easingstyles.Sine, easingdirs.Out, 0.4},
			["rarmweld"] = {CFrame.new(1.26978302, -0.906789303, -0.606788635, 0.819725335, 0.331279457, 0.467229724, 0.410639673, -0.908606768, -0.076213032, 0.399280369, 0.254336864, -0.880844891), easingstyles.Sine, easingdirs.InOut, 0.4},
			["larmweld"] = {CFrame.new(-1.06076431, -1.08841705, -0.702960968, 0.944419086, -0.327317238, -0.0305838138, -0.326595962, -0.92356658, -0.200896934, 0.037510775, 0.199719548, -0.979134798), easingstyles.Sine, easingdirs.InOut, 0.4},
			["headtorootweld"] = {CFrame.Angles(-0.2,0,0) * CFrame.new(-0.000410079956, 0.29989624, 0.00854873657, 0.985827744, -0.000689946115, -0.167757511, -0.00526896119, 0.999370754, -0.0350731388, 0.167676121, 0.0354599804, 0.985204101) * CFrame.new(0,-0.5,0) * CFrame.Angles(-0.15,0,0) * CFrame.new(0,0.5,0), easingstyles.Sine, easingdirs.Out, 0.4},
			["rootweld"] = {CFrame.Angles(-0.3,-0.5,0) * CFrame.new(0, 0, 0, 0.983905256, 0.059391208, 0.168531463, -0.0602797866, 0.998181462, 0.000156628899, -0.168215707, -0.0103131533, 0.985696197):Inverse(), easingstyles.Sine, easingdirs.Out, 0.4},
		},
	},
	["resetkicklegs"] = {
		[1] = {
			["rlegweld"] = {CFrame.new(0.5,-2,0), easingstyles.Sine, easingdirs.Out, 0.2},
			["llegweld"] = {CFrame.new(-0.5,-2,0), easingstyles.Sine, easingdirs.Out, 0.2},
		}
	},
	["attemptcounter"] = {
		[1] = {
			["rarmweld"] = {CFrame.new(0,0,-0.5) * CFrame.new(0.72988987, -0.832155228, -0.852045059, -0.0479691029, 0.484175682, 0.873654902, 0.337815911, -0.815256059, 0.470359445, 0.939988971, 0.317697257, -0.124455139) * CFrame.Angles(0.5,0,0), easingstyles.Cubic, easingdirs.InOut, 0.15},
			["larmweld"] = {CFrame.new(0,0,-0.5) * CFrame.new(-1.05718803, -0.711770535, -0.562652588, -0.580718279, -0.183377504, -0.79318285, -0.302954674, -0.855648756, 0.419623375, -0.755635321, 0.483981431, 0.441335797), easingstyles.Cubic, easingdirs.InOut, 0.15},
			["headtorootweld"] = {CFrame.new(-5.7220459e-05, 0.299463272, -0.0179386139, 0.938507438, -0.00018956096, 0.345258832, -0.020466879, 0.998210728, 0.0561826006, -0.344651699, -0.0597941652, 0.936824262), easingstyles.Cubic, easingdirs.InOut, 0.15},
			["rootweld"] = {CFrame.new(0, 0, 0, 0.939692616, 2.84217094e-14, -0.342020124, 3.15680675e-14, 1, 3.63303866e-15, 0.342020124, 1.42108547e-14, 0.939692616):Inverse(), easingstyles.Cubic, easingdirs.InOut, 0.15},
		}
	},
	["counter"] = { --the animation that costed tears to make it
		[1] = {
			["rlegweld"] = {CFrame.new(0.5,-2,0), easingstyles.Linear, easingdirs.InOut, 0.15},
			["llegweld"] = {CFrame.new(-0.5,-2,0), easingstyles.Linear, easingdirs.Out, 0.15},
			["rarmweld"] = {CFrame.new(0.93245697, -0.257846832, -1.34651184, 0.826473713, 0.319671571, -0.463412285, -0.0404114574, -0.787338555, -0.615194559, -0.561522603, 0.527169406, -0.637796402) * CFrame.Angles(0.3,0,0), easingstyles.Sine, easingdirs.InOut, 0.15},
			["larmweld"] = {CFrame.new(-1.69512558, 0.193106651, -0.635847092, 0.966167808, 0.0933834612, -0.240414277, -0.001156996, -0.930571437, -0.366108298, -0.257911265, 0.35400033, -0.898980141) * CFrame.Angles(0.3,0,0), easingstyles.Sine, easingdirs.InOut, 0.15},
			["headtorootweld"] = {CFrame.new(-5.7220459e-05, 0.299463272, -0.0179367065, 0.938507497, -0.00018956096, 0.345258594, -0.0204668641, 0.998210728, 0.0561826043, -0.344651461, -0.0597941652, 0.936824322) * CFrame.new(0,-0.5,0) * CFrame.Angles(-0.3,0,0) * CFrame.new(0,0.5,0), easingstyles.Sine, easingdirs.Out, 0.15},
			["rootweld"] = {CFrame.new(0, 0, 0, 0.866025269, 8.99846739e-14, -0.500000179, -1.53182402e-14, 1, 3.97505244e-14, 0.500000179, 1.65586979e-15, 0.866025269):Inverse(), easingstyles.Sine, easingdirs.Out, 0.15},
			--
			["rlegweldGUY"] = {CFrame.new(0.5, -1.99999964, -9.53674316e-07, 0.999999762, 1.54226143e-09, -1.86212077e-08, 1.54226143e-09, 0.999999762, -5.96046448e-08, -1.86212077e-08, -5.96046448e-08, 0.999999702), easingstyles.Sine, easingdirs.In, 0.15},
			["llegweldGUY"] = {CFrame.new(-0.5, -1.13969231, -0.457980156, 0.999999762, 7.81808041e-09, -1.69707306e-08, 1.54226143e-09, 0.939692259, 0.342020214, -1.86212077e-08, -0.342020333, 0.939692318), easingstyles.Sine, easingdirs.In, 0.15},
			["rarmweldGUY"] = {CFrame.new(1.88437271, 0.705690622, -1.33960629, 0.86602515, -0.433012575, 0.25000003, 0.0868240222, -0.362167954, -0.928059995, 0.492403656, 0.825429618, -0.276050568), easingstyles.Sine, easingdirs.In, 0.15},
			["larmweldGUY"] = {CFrame.new(-1.68301201, 0.774887323, -1.55896282, 0.86602515, 0.499999881, 1.71470305e-09, 0.0868240744, -0.150383711, -0.984807491, -0.492403746, 0.852868319, -0.17364803), easingstyles.Sine, easingdirs.In, 0.15},
			["headweldGUY"] = {CFrame.new(-1.90734863e-06, 1.49999976, -9.53674316e-07, 0.939692378, 1.54226143e-09, -0.342020065, -1.49011612e-08, 0.999999821, -5.96046448e-08, 0.342020035, -5.96046448e-08, 0.939692378) * CFrame.new(0,-0.5,0) * CFrame.Angles(-0.4,0,0) * CFrame.new(0,0.5,0), easingstyles.Sine, easingdirs.In, 0.15},
			["rootweldGUY"] = {CFrame.new(-0.0694599152, 0.0684039593, -0.387938499, 0.999999881, 0, 2.08616257e-07, 4.09781933e-08, 0.98480767, -0.173648164, -2.23517418e-07, 0.173648119, 0.984807611), easingstyles.Sine, easingdirs.In, 0.15},
		},
		[2] = {
			["rlegweld"] = {CFrame.new(0.500001907, -1.93969202, -0.342020035, 0.999999762, -3.32982353e-09, -1.83858901e-08, 3.1593359e-09, 0.939692318, -0.342020065, -1.84159461e-08, 0.342019975, 0.939692259), easingstyles.Linear, easingdirs.InOut, 0.2},
			["llegweld"] = {CFrame.new(-0.5, -1.69999182, 0.607287407, 0.999999762, 1.61382161e-08, -9.41733447e-09, 3.1593359e-09, 0.642787158, 0.766044557, -1.84159461e-08, -0.766044617, 0.642787099), easingstyles.Linear, easingdirs.Out, 0.2},
			["rarmweld"] = {CFrame.new(1.06463909, -0.641591072, -1.08994675, 0.983257115, 0.0882261321, -0.159439445, -0.00429358426, -0.863514066, -0.50430584, -0.182171077, 0.496546835, -0.848677874), easingstyles.Linear, easingdirs.InOut, 0.2},
			["larmweld"] = {CFrame.new(-1.63196659, -0.151795864, -0.653030396, 0.912136972, -0.0660103187, -0.404534966, -0.225322276, -0.905196607, -0.360345364, -0.342397183, 0.41983521, -0.840536952), easingstyles.Linear, easingdirs.InOut, 0.2},
			["headtorootweld"] = {CFrame.new(-5.7220459e-05, 0.299463272, -0.0179386139, 0.938507438, -0.000189500599, 0.345258594, -0.0204668641, 0.998210728, 0.0561824255, -0.344651461, -0.0597940013, 0.936824262) * CFrame.new(0,-0.5,0) * CFrame.Angles(0.2,0,0) * CFrame.new(0,0.5,0), easingstyles.Linear, easingdirs.Out, 0.2},
			["rootweld"] = {CFrame.new(0, 0, 0, 0.939692497, 8.99846739e-14, -0.342020333, -2.19881232e-14, 1, 3.64866422e-14, 0.342020333, 1.65586979e-15, 0.939692497):Inverse(), easingstyles.Linear, easingdirs.Out, 0.2},
			--
			["rlegweldGUY"] = {CFrame.new(0.500001907, -1.93969202, -0.342020035, 0.999999762, -3.32982353e-09, -1.83858901e-08, 3.1593359e-09, 0.939692318, -0.342020065, -1.84159461e-08, 0.342019975, 0.939692259), easingstyles.Linear, easingdirs.InOut, 0.2},
			["llegweldGUY"] = {CFrame.new(-0.5, -1.69999182, 0.607287407, 0.999999762, 1.61382161e-08, -9.41733447e-09, 3.1593359e-09, 0.642787158, 0.766044557, -1.84159461e-08, -0.766044617, 0.642787099), easingstyles.Linear, easingdirs.Out, 0.2},
			["rarmweldGUY"] = {CFrame.new(1.9939518, 0.424452662, -0.622869492, 0.777676404, -0.627536476, 0.0376398899, 0.0226151794, -0.0319085345, -0.999234617, 0.628257275, 0.777932644, -0.0106225535) * CFrame.new(0,0.5,0) * CFrame.Angles(0,0,0.5) * CFrame.new(0,-0.5,0), easingstyles.Linear, easingdirs.InOut, 0.2},
			["larmweldGUY"] = {CFrame.new(-1.62628174, 0.605995059, -0.601126671, 0.965925574, 0.258818984, 8.65547634e-11, 0.044943437, -0.167731196, -0.984807491, -0.254887015, 0.951250792, -0.17364803) * CFrame.new(0,0.5,0) * CFrame.Angles(0.3,0,-0.9) * CFrame.new(0,-0.5,0), easingstyles.Sine, easingdirs.Out, 0.2},
			["headweldGUY"] = {CFrame.new(0, 1.49999964, 0, 0.999999762, 3.1593359e-09, 2.20002605e-07, 3.15934656e-09, 0.999999821, -4.47034836e-08, -2.56834483e-07, -4.47034836e-08, 0.999999702), easingstyles.Linear, easingdirs.Out, 0.2},
			["rootweldGUY"] = {CFrame.new(0,0,0.25) * CFrame.new(-0.0694599152, 0.0684039593, 0.0120620728, 0.999999881, -1.81821029e-08, 2.07822396e-07, 4.09781933e-08, 0.996194601, -0.087155737, -2.23517418e-07, 0.0871556997, 0.996194541), easingstyles.Linear, easingdirs.Out, 0.2},
		},
		[3] = {
			["rlegweld"] = {CFrame.new(0.500001907, -1.93969202, -0.342020035, 0.999999762, -3.32982353e-09, -1.83858901e-08, 3.1593359e-09, 0.939692318, -0.342020065, -1.84159461e-08, 0.342019975, 0.939692259) * CFrame.new(0,1,0) * CFrame.Angles(-0.4,0,0) * CFrame.new(0,-1,0), easingstyles.Linear, easingdirs.InOut, 0.15},
			["llegweld"] = {CFrame.new(-0.5, -1.69999182, 0.607287407, 0.999999762, 1.61382161e-08, -9.41733447e-09, 3.1593359e-09, 0.642787158, 0.766044557, -1.84159461e-08, -0.766044617, 0.642787099) * CFrame.new(0,1,0) * CFrame.Angles(0.4,0,0) * CFrame.new(0,-1,0), easingstyles.Linear, easingdirs.Out, 0.15},
			["rarmweld"] = {CFrame.new(0.750505447, -0.598126888, -1.17032242, 0.958039403, 0.229040861, -0.172337487, 0.15864794, -0.924462795, -0.346696615, -0.238727212, 0.304807991, -0.922008991), easingstyles.Linear, easingdirs.InOut, 0.15},
			["larmweld"] = {CFrame.new(-1.39134216, -0.695871353, -0.835510254, 0.968526304, -0.066010341, -0.23999843, -0.159325898, -0.905196548, -0.393997669, -0.191237807, 0.41983515, -0.88722384), easingstyles.Linear, easingdirs.InOut, 0.15},
			["headtorootweld"] = {CFrame.new(-5.7220459e-05, 0.299463272, -0.0179386139, 0.938507318, -0.000189500599, 0.345258564, -0.0204668604, 0.998210728, 0.056182418, -0.344651431, -0.0597940013, 0.936824262), easingstyles.Linear, easingdirs.Out, 0.15},
			["rootweld"] = {CFrame.new(0, 0, 0, 0.984807611, 8.99846739e-14, -0.173648372, -2.79899158e-14, 1, 3.21141293e-14, 0.173648372, 1.65586979e-15, 0.984807611):Inverse(), easingstyles.Linear, easingdirs.Out, 0.15},
			--
			["rlegweldGUY"] = {CFrame.new(0.5, -1.46338248, -0.324464798, 0.999999762, 7.4505806e-09, -2.98023224e-08, 1.86264515e-09, 0.984807491, 0.173648119, -4.47034836e-08, -0.173648119, 0.984807312), easingstyles.Linear, easingdirs.InOut, 0.15},
			["llegweldGUY"] = {CFrame.new(-0.406030655, -1.96145964, 0.382390976, 0.996194482, -0.0871557072, -2.98023224e-08, 0.0818995833, 0.936116636, 0.342020661, -0.0298091173, -0.340719104, 0.939692199), easingstyles.Linear, easingdirs.Out, 0.15},
			["rarmweldGUY"] = {CFrame.new(1.99394989, 0.424452782, -0.622869492, 0.777676404, -0.627536476, 0.0376398936, 0.0226152092, -0.0319085121, -0.999234736, 0.628257155, 0.777932584, -0.0106225908) * CFrame.Angles(0,0,-0.5) * CFrame.new(0.25,0,0), easingstyles.Linear, easingdirs.InOut, 0.15},
			["larmweldGUY"] = {CFrame.new(0,0,0.25) * CFrame.new(-1.51391411, 0.588179111, -0.751041412, 0.999999642, -2.98023224e-08, 2.57424659e-09, -1.3038516e-08, -0.0871556923, -0.996194601, -1.49011612e-07, 0.996194243, -0.0871556774) * CFrame.Angles(-0.3,0,0.3) * CFrame.new(0,-0.25,0), easingstyles.Sine, easingdirs.In, 0.15},
			["headweldGUY"] = {CFrame.new(-1.90734863e-06, 1.5, 0, 0.984807432, 1.86264515e-09, 0.173648313, 3.7252903e-09, 0.999999881, 0, -0.173648387, 0, 0.984807312), easingstyles.Linear, easingdirs.Out, 0.15},
			["rootweldGUY"] = {CFrame.new(0,0,0.5) * CFrame.Angles(0,-0.2,0) * CFrame.new(-0.0694599152, 0.0684039593, 0.212061882, 0.965925872, 0.0225574747, -0.25783354, -1.05551807e-08, 0.996194661, 0.0871555358, 0.258818388, -0.0841858089, 0.962250173), easingstyles.Linear, easingdirs.Out, 0.15},
		},
		[4] = {
			["rlegweld"] = {CFrame.new(0.500001907, -1.93969202, -0.342020035, 0.999999762, -3.32982353e-09, -1.83858901e-08, 3.1593359e-09, 0.939692318, -0.342020065, -1.84159461e-08, 0.342019975, 0.939692259) * CFrame.new(0,1,0) * CFrame.Angles(-0.6,0,0) * CFrame.new(0,-1,0), easingstyles.Linear, easingdirs.InOut, 0.15},
			["llegweld"] = {CFrame.new(-0.5, -1.69999182, 0.607287407, 0.999999762, 1.61382161e-08, -9.41733447e-09, 3.1593359e-09, 0.642787158, 0.766044557, -1.84159461e-08, -0.766044617, 0.642787099) * CFrame.new(0,1,0) * CFrame.Angles(0.6,0,0) * CFrame.new(0,-1,0), easingstyles.Linear, easingdirs.Out, 0.15},
			["rarmweld"] = {CFrame.new(0.687705994, -0.724755764, -1.11096668, 0.903712034, 0.391922951, -0.172337502, 0.316769004, -0.882869184, -0.346696615, -0.28802973, 0.258722663, -0.922008932), easingstyles.Linear, easingdirs.InOut, 0.15},
			["larmweld"] = {CFrame.new(-1.19763565, -0.727736473, -0.873757362, 0.968526185, -0.066010341, -0.2399984, -0.159325898, -0.905196548, -0.393997639, -0.191237807, 0.41983512, -0.88722384), easingstyles.Linear, easingdirs.InOut, 0.15},
			["headtorootweld"] = {CFrame.new(-5.7220459e-05, 0.299463272, -0.0179376602, 0.938507378, -0.000189500584, 0.345258534, -0.0204668604, 0.998210728, 0.0561824217, -0.344651431, -0.0597940013, 0.936824203), easingstyles.Linear, easingdirs.Out, 0.15},
			["rootweld"] = {CFrame.new(0, 0, 0, 0.992298067, 8.99846739e-14, 0.12387231, -3.62015444e-14, 1, 2.24544064e-14, -0.12387231, 1.65586979e-15, 0.992298067):Inverse() * CFrame.new(0.25,0,-0.25), easingstyles.Linear, easingdirs.Out, 0.15},
			--
			["rlegweldGUY"] = {CFrame.new(0.859134674, -1.87342918, -0.297487259, 0.982262433, -0.187177643, 0.011183247, 0.186983958, 0.982226014, 0.0163999647, -0.0140541941, -0.0140179824, 0.999803007), easingstyles.Cubic, easingdirs.Out, 0.35},
			["llegweldGUY"] = {CFrame.new(-0.802511215, -2.16130018, 0.180358887, 0.952584267, 0.287517309, 0.0995830446, -0.298857421, 0.94557929, 0.128700078, -0.0571602136, -0.152358815, 0.986671031) * CFrame.new(-0.35,0.5,-0.4), easingstyles.Cubic, easingdirs.Out, 0.35},
			["rarmweldGUY"] = {CFrame.new(0,0,0.25) * CFrame.new(2.24226809, 0.280616283, -0.925067902, 0.777676642, -0.624539077, -0.0719023943, 0.0226151943, 0.142091364, -0.989595115, 0.628257453, 0.767958999, 0.124625243) * CFrame.Angles(0,0,-0.5), easingstyles.Sine, easingdirs.Out, 0.35},
			["larmweldGUY"] = {CFrame.new(-0.378030777, 0.471939564, -0.942680359, 0.317914963, -0.886304021, -0.336741805, -0.940296233, -0.340266466, 0.00785556436, -0.121544391, 0.314139694, -0.941564202), easingstyles.Cubic, easingdirs.Out, 0.3},
			["headweldGUY"] = {CFrame.new(-0.0731220245, 1.49126625, -0.0575294495, 0.360467374, -0.146244928, 0.921235919, -0.0553840958, 0.982534528, 0.177647054, -0.931126118, -0.115057789, 0.346072018), easingstyles.Cubic, easingdirs.Out, 0.3},
			["rootweldGUY"] = {CFrame.new(0,-0.5,0.75) * CFrame.Angles(-0.3,-0.6,0) * CFrame.new(-0.0694561005, 0.0684034824, 0.812059402, 0.0871563852, 0.0868238881, -0.992403865, -0.17298761, 0.982379317, 0.0707544759, 0.981060147, 0.165506855, 0.100640081), easingstyles.Sine, easingdirs.Out, 0.3},
		},
		[5] = {
			["rlegweld"] = {CFrame.new(1.27769852, -1.60626066, 0.810951233, 0.813813329, -0.195074677, -0.547406197, 0.338103056, 0.925073922, 0.172987193, 0.472645819, -0.32585901, 0.81879276), easingstyles.Linear, easingdirs.InOut, 0.15},
			["llegweld"] = {CFrame.new(-0.176330566, -1.92388415, 0.124156952, 0.836572409, 0.0731905848, -0.542945385, -0.087155737, 0.99619478, 3.69733676e-14, 0.540879309, 0.047320798, 0.839767873), easingstyles.Linear, easingdirs.Out, 0.15},
			["rarmweld"] = {CFrame.new(0.588619232, -0.759009838, -1.00744629, 0.112379178, 0.97824955, 0.174349785, 0.94360733, -0.0500737354, -0.327256739, -0.311408281, 0.201294571, -0.928711057), easingstyles.Linear, easingdirs.InOut, 0.15},
			["larmweld"] = {CFrame.new(-1.62687778, -0.461441517, -1.04625511, -0.686910808, 0.386318982, -0.615557075, -0.726315141, -0.393936783, 0.563275933, -0.0248862207, 0.834008932, 0.551188827), easingstyles.Linear, easingdirs.InOut, 0.15},
			["headtorootweld"] = {CFrame.new(-5.62667847e-05, 0.299463272, -0.0179386139, 0.419490278, -0.000189483166, 0.907759547, -0.054199215, 0.998210788, 0.0252547339, -0.906140089, -0.059794005, 0.418729335), easingstyles.Linear, easingdirs.Out, 0.15},
			["rootweld"] = {CFrame.new(0.22731781, 0, 0.385131836, 0.836572349, -0.0731905848, -0.542945385, 0.087155737, 0.99619472, 4.04709835e-14, 0.540879309, -0.0473208055, 0.839767873):Inverse() * CFrame.new(0.5,0,0), easingstyles.Linear, easingdirs.Out, 0.15},
			--
			["rlegweldGUY"] = {CFrame.new(0.66415596, -2.07281971, -0.0919952393, 0.982262373, -0.187177539, 0.0111834109, 0.186983988, 0.982225895, 0.0163995475, -0.0140542611, -0.0140175484, 0.999802828), easingstyles.Sine, easingdirs.In, 0.15},
			["llegweldGUY"] = {CFrame.new(-0.438922882, -2.03577328, 0.127288818, 0.993473172, -0.0556249209, 0.0995831117, 0.042573195, 0.990769029, 0.128700078, -0.105822787, -0.123620503, 0.986670732), easingstyles.Sine, easingdirs.In, 0.15},
			["rarmweldGUY"] = {CFrame.new(2.51877975, 0.309803963, -0.533771515, 0.194289014, -0.978305519, -0.0719024986, 0.10865882, 0.0943114012, -0.989594936, 0.974907458, 0.18445462, 0.124625236) * CFrame.Angles(0,0,0), easingstyles.Sine, easingdirs.In, 0.15},
			["larmweldGUY"] = {CFrame.new(-0.37802887, 0.471940994, -0.942679405, 0.317914873, -0.88630414, -0.336741865, -0.940296054, -0.340266496, 0.00785551593, -0.121544354, 0.314139664, -0.941564083), easingstyles.Sine, easingdirs.In, 0.15},
			["headweldGUY"] = {CFrame.new(-0.0731220245, 1.4912653, -0.0575284958, 0.98896575, -0.146244809, -0.0236472338, 0.14799121, 0.982534409, 0.112802871, 0.00673744082, -0.115057781, 0.993335903), easingstyles.Sine, easingdirs.In, 0.15},
			["rootweldGUY"] = {CFrame.new(0,0,-0.25) * CFrame.new(-0.10982132, 0.0683119297, 0.414100647, 0.908486545, 0.405548543, 0.100909129, -0.407650352, 0.913137972, 0.000228819132, -0.0920511708, -0.0413435362, 0.994895577), easingstyles.Sine, easingdirs.In, 0.15},
		},
		[6] = {
			["rlegweld"] = {CFrame.new(0.986347198, -1.91204035, -0.290897369, 0.970146358, -0.239886254, -0.0356335938, 0.241891384, 0.967695594, 0.0710914433, 0.0174286067, -0.0775884315, 0.996832907), easingstyles.Linear, easingdirs.InOut, 0.125},
			["llegweld"] = {CFrame.new(-0.716581345, -1.63094723, 0.0695590973, 0.960399628, 0.270133168, 0.0682640076, -0.277699918, 0.908086419, 0.313467175, 0.0226882398, -0.320010573, 0.947141886), easingstyles.Linear, easingdirs.Out, 0.125},
			["rarmweld"] = {CFrame.new(0,0.25,0) * CFrame.new(0.315651894, -0.74064362, -1.35473251, 0.370858222, 0.675225139, 0.637600005, 0.89553082, -0.0781941861, -0.43807447, -0.245942146, 0.733454466, -0.633684456) * CFrame.new(0,0.5,0) * CFrame.Angles(0.25,-0.5,-0.25) * CFrame.Angles(0.25,0,0.25) * CFrame.new(0,-0.5,0), easingstyles.Linear, easingdirs.InOut, 0.125},
			["larmweld"] = {CFrame.new(-2.13316631, -0.42100811, -0.753929138, 0.215355352, 0.49663341, -0.840818763, -0.807522476, -0.393595427, -0.4393062, -0.549116731, 0.773587227, 0.316279948) * CFrame.new(0,0.5,0) * CFrame.Angles(0.5,0,0) * CFrame.new(0,-0.5,0), easingstyles.Linear, easingdirs.InOut, 0.125},
			["headtorootweld"] = {CFrame.new(0.0237092972, 0.293178797, -0.0690860748, 0.416809887, 0.0473450199, 0.907759309, -0.166959584, 0.985640168, 0.0252547115, -0.8935287, -0.162085533, 0.418729305), easingstyles.Linear, easingdirs.Out, 0.125},
			["rootweld"] = {CFrame.new(0.496259689, -0.0153713226, 0.778461456, 0.321300089, 0.142697498, -0.9361642, -0.040174026, 0.989745438, 0.137076691, 0.946124673, -0.0064332746, 0.323738039):Inverse() * CFrame.new(1,0,0), easingstyles.Linear, easingdirs.Out, 0.125},
			--
			["rlegweldGUY"] = {CFrame.new(0.611109734, -1.73090172, 0.749567032, 0.982262313, -0.167692289, -0.0839037299, 0.186983898, 0.842432559, 0.505315602, -0.0140542462, -0.512041092, 0.858846009), easingstyles.Linear, easingdirs.In, 0.125},
			["llegweldGUY"] = {CFrame.new(-0.615002632, -2.11446095, 0.155323029, 0.988039136, 0.117734872, 0.0995830595, -0.130118906, 0.983109653, 0.128700152, -0.0827486515, -0.140118256, 0.986670971), easingstyles.Linear, easingdirs.In, 0.125},
			["rarmweldGUY"] = {CFrame.new(0,-0.25,0) * CFrame.new(2.39652157, 0.875180244, -0.429607391, 0.194288969, -0.943898439, 0.267034233, 0.10865888, -0.249837741, -0.962171316, 0.974907577, 0.215955019, 0.0540220439) * CFrame.Angles(-0.3,0,0.3), easingstyles.Linear, easingdirs.In, 0.125},
			["larmweldGUY"] = {CFrame.new(-0.378030777, 0.471939087, -0.942680359, 0.317914903, -0.886303961, -0.336741775, -0.940295875, -0.340266317, 0.00785542652, -0.121544436, 0.314139664, -0.941564262), easingstyles.Linear, easingdirs.In, 0.125},
			["headweldGUY"] = {CFrame.new(-0.0731229782, 1.49126434, -0.0575294495, 0.474003732, -0.146244913, -0.868293047, 0.171685785, 0.982534289, -0.0717625692, 0.863622963, -0.115057699, 0.490833163), easingstyles.Linear, easingdirs.In, 0.125},
			["rootweldGUY"] = {CFrame.new(0,0,-0.5) * CFrame.new(-0.10982132, 0.0683119297, 0.414100647, -0.100909166, 0.405548513, 0.908486545, -0.000228801306, 0.913137913, -0.407650352, -0.994895577, -0.0413435325, -0.0920512155), easingstyles.Linear, easingdirs.In, 0.125},
		},
		[7] = {
			["rlegweld"] = {CFrame.new(0.802715302, -1.74061966, -0.0724935532, 0.939692616, -0.342020154, -5.96046448e-08, 0.336823881, 0.925416172, 0.173648015, -0.0593912601, -0.163175941, 0.984807611), easingstyles.Linear, easingdirs.InOut, 0.125},
			["llegweld"] = {CFrame.new(-0.837442398, -1.85863495, -0.312516212, 0.939692616, 0.342020094, -2.98023224e-08, -0.321393698, 0.883021772, -0.342020094, -0.116977736, 0.321393669, 0.939692616), easingstyles.Linear, easingdirs.Out, 0.125},
			["rarmweld"] = {CFrame.new(0.936542511, -0.698741674, -1.23487282, -0.00559486449, 0.407685161, 0.913105428, 0.998487592, -0.047657568, 0.0273961537, 0.0546852387, 0.911877871, -0.406802028), easingstyles.Linear, easingdirs.InOut, 0.125},
			["larmweld"] = {CFrame.new(-0.5311203, -0.690046787, -1.44159126, 0.0895578712, -0.579664946, -0.809918582, -0.995722234, -0.0706537962, -0.0595357493, -0.0227129199, 0.811786056, -0.583513081), easingstyles.Linear, easingdirs.InOut, 0.125},
			["headtorootweld"] = {CFrame.new(0.00373840332, 0.297858953, -0.0355796814, 0.718141854, 0.0124605056, 0.695785165, -0.0914169624, 0.992863894, 0.0765734166, -0.689866066, -0.118597277, 0.71415633), easingstyles.Linear, easingdirs.Out, 0.125},
			["rootweld"] = {CFrame.new(-0.5,0,-0.5) * CFrame.Angles(0,-0.3,0) * CFrame.new(0, 0, 0, -0.34345296, 0.242945567, -0.907203078, 0.104686886, 0.96984607, 0.220088273, 0.933317065, -0.0193823203, -0.358529806):Inverse(), easingstyles.Linear, easingdirs.Out, 0.125},
			--
			["rlegweldGUY"] = {CFrame.new(0.479580879, -1.90857697, 0.176668167, 0.984490633, 0.173830897, -0.0236841142, -0.138055414, 0.850928843, 0.506813705, 0.108253092, -0.495683581, 0.861730099), easingstyles.Linear, easingdirs.In, 0.125},
			["llegweldGUY"] = {CFrame.new(-1.18886852, -2.15475273, 0.218500137, 0.79679966, 0.595981061, 0.0995829701, -0.604241014, 0.786338449, 0.128700182, -0.00160339475, -0.162720352, 0.986670852), easingstyles.Linear, easingdirs.In, 0.125},
			["rarmweldGUY"] = {CFrame.new(2.47537899, 0.949079514, 0.496810913, -0.598181784, -0.755560219, 0.267034233, -0.121542238, -0.243830249, -0.962171376, 0.792089581, -0.608009219, 0.0540219843), easingstyles.Linear, easingdirs.In, 0.125},
			["larmweldGUY"] = {CFrame.new(-0.589935303, 0.434659958, -1.20854378, 0.317914903, -0.46249488, -0.827664435, -0.940295935, -0.265708476, -0.212701485, -0.12154451, 0.845870733, -0.519354701), easingstyles.Linear, easingdirs.In, 0.125},
			["headweldGUY"] = {CFrame.new(-0.0731229782, 1.49126434, -0.0575256348, 0.474003613, -0.146244854, -0.868293226, 0.171685874, 0.982534289, -0.0717626065, 0.863622904, -0.11505764, 0.490833312), easingstyles.Linear, easingdirs.In, 0.125},
			["rootweldGUY"] = {CFrame.new(-0.5,0,0) * CFrame.new(-1.76003265, 0.084195137, -2.73905277, -0.664035678, 0.318418264, 0.676510572, -0.0805087537, 0.869077861, -0.488079607, -0.743353903, -0.378567368, -0.551463246) * CFrame.new(-2.5,-0.2,0.3), easingstyles.Linear, easingdirs.In, 0.125},
		},
		-------------------------
		[8] = {
			["rlegweld"] = {CFrame.new(0.802713394, -1.74061871, -0.0724945068, 0.939692557, -0.342020154, 0, 0.336823851, 0.925416231, 0.173648059, -0.0593912303, -0.163175911, 0.984807611), easingstyles.Linear, easingdirs.InOut, 0.18},
			["llegweld"] = {CFrame.new(-0.837446213, -1.85863447, -0.312517166, 0.939692616, 0.342020035, -2.98023224e-08, -0.321393698, 0.883021653, -0.342020065, -0.116977811, 0.321393698, 0.939692497), easingstyles.Linear, easingdirs.Out, 0.18},
			["rarmweld"] = {CFrame.new(1.72044182, -0.703189492, -1.07345772, -0.00559484959, -0.103486955, 0.994615138, 0.998487532, -0.0549707189, -0.00010304898, 0.0546852574, 0.993110418, 0.103638023), easingstyles.Linear, easingdirs.InOut, 0.18},
			["larmweld"] = {CFrame.new(-0.31354332, -0.756659269, -1.30835724, -0.0124604804, -0.9128052, -0.408205152, -0.992863834, 0.0597142167, -0.103221945, 0.118597277, 0.404006124, -0.907035589), easingstyles.Linear, easingdirs.InOut, 0.18},
			["headtorootweld"] = {CFrame.new(0.00373840332, 0.297859311, -0.0355787277, 0.718141794, 0.0124604963, 0.695785284, -0.0914169773, 0.992863834, 0.0765734091, -0.689866126, -0.118597277, 0.714156389), easingstyles.Linear, easingdirs.Out, 0.18},
			["rootweld"] = {CFrame.new(-0.5,0,0) * CFrame.new(0, 0, 0, -0.63302213, 0.242945567, -0.735024095, 0.173648164, 0.96984607, 0.171010256, 0.754406631, -0.0193823203, -0.656121194):Inverse(), easingstyles.Linear, easingdirs.Out, 0.18},
			--
			["rlegweldGUY"] = {CFrame.new(0.333839417, -1.79884243, 0.108122826, 0.93934834, 0.342144787, -0.0236839056, -0.283720255, 0.814028263, 0.506813526, 0.19268325, -0.469355077, 0.86172998), easingstyles.Linear, easingdirs.In, 0.18},
			["llegweldGUY"] = {CFrame.new(-1.5513916, -2.01691914, 0.237108231, 0.544909, 0.832560301, 0.0995831788, -0.836744547, 0.532253921, 0.128700048, 0.0541471988, -0.153455466, 0.986670852), easingstyles.Linear, easingdirs.In, 0.18},
			["rarmweldGUY"] = {CFrame.new(2.02225494, 1.20856571, 0.426721573, -0.359039009, -0.718164504, 0.596095383, -0.340167999, -0.494057298, -0.800120234, 0.869123757, -0.490047157, -0.0669108629), easingstyles.Linear, easingdirs.In, 0.18},
			["larmweldGUY"] = {CFrame.new(-1.77902222, 0.195022583, -0.819361687, 0.317914724, 0.485531092, -0.814364195, -0.940295815, 0.0513504967, -0.336461127, -0.121544272, 0.872709811, 0.472868174), easingstyles.Linear, easingdirs.In, 0.18},
			["headweldGUY"] = {CFrame.new(-0.0731220245, 1.49126625, -0.0575284958, 0.474003732, -0.146244749, -0.868292928, 0.171685696, 0.982534289, -0.0717627406, 0.863622963, -0.115057856, 0.490833074), easingstyles.Linear, easingdirs.In, 0.18},
			["rootweldGUY"] = {CFrame.new(0.25,0,0) * CFrame.new(-2.59634018, 0.0384702682, -4.0757637, 0.0914031863, 0.318418264, 0.943533361, -0.425640643, 0.869077861, -0.252058268, -0.900263906, -0.378567368, 0.214968324) * CFrame.Angles(0,-0.2,0), easingstyles.Linear, easingdirs.In, 0.18},
		},
		-------------------------
		[9] = {
			["rlegweld"] = {CFrame.new(0.630540848, -1.82856011, -0.0569896698, 0.999999642, -6.70552254e-08, 0, 1.86264515e-09, 0.984807312, 0.173648074, -2.98023224e-08, -0.173648208, 0.984807789), easingstyles.Sine, easingdirs.Out, 0.2},
			["llegweld"] = {CFrame.new(-0.816818237, -1.92236018, 0.02825737, 0.939692378, 0.321393669, 0.116977721, -0.321393698, 0.946746707, -0.0193824768, -0.116977811, -0.019382447, 0.992945373), easingstyles.Sine, easingdirs.Out, 0.2},
			["rarmweld"] = {CFrame.new(2.08691597, -1.02805924, -0.51381588, 0.288619757, -0.643917799, 0.708567619, 0.888492882, 0.455889642, 0.0523857102, -0.356761009, 0.614437997, 0.703695416), easingstyles.Sine, easingdirs.Out, 0.2},
			["larmweld"] = {CFrame.new(-0.831960678, -1.04188442, -1.75698471, 0.0290367156, -0.298992932, -0.953813374, -0.878030658, 0.448412597, -0.16729404, 0.47772193, 0.842335582, -0.249504685), easingstyles.Sine, easingdirs.Out, 0.2},
			["headtorootweld"] = {CFrame.new(0.00373649597, 0.297858596, -0.0355787277, 0.961638272, 0.0124605205, -0.274036556, 0.0206060931, 0.992863894, 0.11745616, 0.273544669, -0.118597239, 0.954519808), easingstyles.Sine, easingdirs.Out, 0.2},
			["rootweld"] = {CFrame.new(-0.5,0,0) * CFrame.new(-0.188798904, 0.0127785206, 0.0647411346, -0.944000244, 0.0802406743, -0.320038766, 0.0638931543, 0.996073604, 0.0612752065, 0.323698968, 0.0373955444, -0.945420861):Inverse(), easingstyles.Sine, easingdirs.Out, 0.2},
			--
			["rlegweldGUY"] = {CFrame.new(0.644073486, -1.79823875, 0.474986076, 0.999701917, 0.00173184276, -0.0243404694, 0.0170732588, 0.663026094, 0.748401225, 0.0174348503, -0.748593628, 0.662798822), easingstyles.Linear, easingdirs.In, 0.15},
			["llegweldGUY"] = {CFrame.new(-1.03019333, -1.78052139, 0.915860176, 0.860481739, 0.469148427, 0.198670849, -0.441927224, 0.493264496, 0.74925971, 0.253517151, -0.732522309, 0.631774664), easingstyles.Linear, easingdirs.In, 0.15},
			["rarmweldGUY"] = {CFrame.new(1.91228676, 1.12462616, 0.450775146, -0.493869424, -0.633058071, 0.596095443, -0.431086898, -0.417098314, -0.800120354, 0.755152941, -0.652124047, -0.0669106841), easingstyles.Linear, easingdirs.In, 0.15},
			["larmweldGUY"] = {CFrame.new(-2.08757591, 0.22996521, -0.505936623, 0.317914784, 0.775287271, -0.545764685, -0.940295994, 0.184024692, -0.286317736, -0.121544242, 0.604205191, 0.787503719), easingstyles.Linear, easingdirs.In, 0.15},
			["headweldGUY"] = {CFrame.new(-0.0731239319, 1.49126434, -0.0575284958, 0.831632495, -0.146244898, -0.535723269, 0.185095727, 0.982534349, 0.0191161633, 0.523571134, -0.115057737, 0.84417665), easingstyles.Linear, easingdirs.In, 0.15},
			["rootweldGUY"] = {CFrame.new(0.7,0,0) * CFrame.new(-2.14434242, -0.0581073761, -6.17290115, 0.742119312, 0.29003197, 0.604268312, -0.60326153, 0.681932151, 0.413574338, -0.292120159, -0.671453536, 0.681040108) * CFrame.Angles(0,0,0.5), easingstyles.Linear, easingdirs.In, 0.15},
		},
	}
}
tween = function(speed, easingstyle, easingdirection, loopcount, WHAT, goal)
	local info = TweenInfo.new(
		speed,
		easingstyle,
		easingdirection,
		loopcount
	)
	local goals = goal
	local anim = tweenservice:Create(WHAT, info, goals)
	anim:Play()
end
pose = function(posename, index, speed, senttick)
	if senttick == currentanimtick then
		for i,v in pairs(animrawdata[posename][index]) do
			if currentwelds[i] then
				tween(
					v[4]*speed,
					v[2],
					v[3],
					0,
					currentwelds[i],
					{C0 = v[1]}
				)
			end
		end
	end
end
parentfindfirstchildofclass = function(cname, search)
	local par = search
	local foundinstance
	while par ~= workspace and not foundinstance do
		foundinstance = par:FindFirstChildOfClass(cname)
		par = par.Parent
	end
	return foundinstance
end
playsound = function(name, speed, timepos)
	local thesound = currentsounds[name]
	if not thesound then
		print(name, "sound not found")
		return nil
	end
	thesound.PlaybackSpeed = speed or 1
	thesound.TimePosition = timepos or 0
	thesound:Play()
end
playremovesound = function(name, parent, speed, timepos)
	local thesound = currentsounds[name]
	if not thesound then
		print(name, "sound not found")
		return nil
	end
	thesound.Parent = parent
	thesound.PlaybackSpeed = speed
	thesound.TimePosition = timepos or 0
	thesound.Parent = nil
end
makeplayonremovesound = function(id, volume, rmax, rmin, speed, name)
	local s = Instance.new("Sound")
	s.SoundId = "rbxassetid://"..id
	s.Name = name
	s.Volume = volume or 1
	s.RollOffMaxDistance = rmax or 1e4
	s.RollOffMinDistance = rmin or 10
	s.PlaybackSpeed = speed or 1
	currentsounds[s.Name] = s
	s.Parent = nil
	s.PlayOnRemove = true
end
equipped = function()
	return tool.Parent == character
end
makesound = function(id, parent, volume, rmax, rmin, speed, name)
	local s = Instance.new("Sound")
	s.SoundId = "rbxassetid://"..id
	s.Name = name
	s.Volume = volume or 1
	s.RollOffMaxDistance = rmax or 1e4
	s.RollOffMinDistance = rmin or 10
	s.PlaybackSpeed = speed or 1
	s.Parent = parent
	if sfxbehavior[name] then
		sfxbehavior[name](s)
	end
	currentsounds[s.Name] = s
end
makepart = function(parent, size, cf, anchored, cancol, name) --spawnlocation because spawns have less limit on vsb
	local part = Instance.new("SpawnLocation")
	part.Neutral = false
	part.Enabled = false
	part.Anchored = anchored
	part.CanCollide = cancol
	part.Name = name or "Part"
	part.Size = size
	part.CFrame = cf
	part.Parent = parent
	part:BreakJoints()
	return part
end
weld = function(part0, part1, c0, name)
	local w = Instance.new("Weld")
	w.Part0 = part0
	w.Part1 = part1
	w.C0 = c0
	w.Name = name
	w.Parent = part0
	currentwelds[w.Name] = w
end
removeweld = function(weldname)
	local foundweld = currentwelds[weldname]
	if foundweld then
		foundweld:Destroy()
		foundweld = nil
	else
		print(weldname, "not found")
	end
end
unweldasdescendantof = function(desc) --minor spelling mistake
	for i,v in pairs(currentwelds) do
		if v:IsDescendantOf(desc) then
			removeweld(i)
		end
	end
end
faketorso = function(npc) --do it like its 2012
	local amodel = Instance.new("Model", npc)
	amodel.Name = "torsoholder"
	local modelhum = Instance.new("Humanoid")
	modelhum.RequiresNeck = false
	modelhum.MaxHealth = tonumber("nan")
	modelhum.Health = modelhum.MaxHealth
	local fakepart = Instance.new("Part")
	fakepart.Size = Vector3.new(2,2,1)
	fakepart.CanCollide = false
	fakepart.Name = "Torso"
	fakepart.CFrame = npc.Torso.CFrame
	fakepart.Color = npc.Torso.Color
	fakepart.Parent = amodel
	npc.PrimaryPart = fakepart
	local fakeweld = Instance.new("Weld", fakepart) --but its real how can it be fake
	fakeweld.Part0 = npc.Torso
	fakeweld.Part1 = fakepart
	fakeweld.Name = "RootJoint"
	modelhum.Parent = amodel
	for i,v in pairs(npc:GetChildren()) do
		if (v:IsA("CharacterMesh") and v.BodyPart.Name == "Torso") or v:IsA("Shirt") or v:IsA("ShirtGraphic") then
			v:Clone().Parent = amodel
		end
	end
end
local notable_nerve_response = { --the code where it makes you DIE
	["Head"] = function(part, thehum)
		remote:FireClient(owner, "showredcross", 0.5)
		thehum.Health = thehum.Health - (DAMAGE*0.15) --100% base and + 15% more dmg to the head
		playremovesound("snap", part, 1+math.random(-20,20)/70, 0)
		playremovesound("bone", part, 1+math.random(-20,20)/70, 0)
		playremovesound("hitflesh"..math.random(1,3), part, 1+math.random(-20,20)/70, 0)
		if not thehum.PlatformStand and thehum:GetState() ~= Enum.HumanoidStateType.FallingDown then
			thehum.PlatformStand = true
			part.CFrame = CFrame.lookAt(part.Position, Vector3.new(charhead.Position.x,part.Position.y,charhead.Position.z))
			local vel = Instance.new("BodyAngularVelocity", part) --problem officer?
			vel.MaxTorque = Vector3.new(1/0,1/0,1/0)
			vel.AngularVelocity = (charroot.CFrame.LookVector * hitvector[swingdatas[swinganimation][1]]*4) + (charroot.CFrame.UpVector * -hitvector[swingdatas[swinganimation][1]]*13)
			debris:AddItem(vel, 0.1)
			if toolstate ~= "leftkicking" and toolstate ~= "rightkicking" then
				local vel = Instance.new("BodyVelocity", part) --problem officer?
				vel.MaxForce = Vector3.new(1/0,1/0,1/0)
				vel.Velocity = charroot.CFrame.LookVector * 10
				debris:AddItem(vel, 0.2)
			end
			task.spawn(function()
				task.wait(1.5)
				thehum.PlatformStand = false
			end)
		end
	end,
}
local notable_state_response = {
	["leftkicking"] = function(part, thehum) --kopnięcie szybkości czasu
		if thehum and thehum:IsA("Humanoid") then
			thehum.Health = thehum.Health - (DAMAGE*0.25) --100% base and + 25% more dmg
			playremovesound("bash", part, 0.5+math.random(0,20)/70, 0)
			playremovesound("hitflesh"..math.random(1,3), part, 1+math.random(-20,20)/70, 0)
			local vel = Instance.new("BodyVelocity", part) --problem officer?
			vel.MaxForce = Vector3.new(1/0,1/0,1/0)
			vel.Velocity = charroot.CFrame.LookVector * 12.5
			debris:AddItem(vel, 0.2)
			if not thehum.PlatformStand and thehum:GetState() ~= Enum.HumanoidStateType.FallingDown then
				part.CFrame = CFrame.lookAt(part.Position, Vector3.new(charhead.Position.x,part.Position.y,charhead.Position.z))
				thehum.PlatformStand = true
				task.spawn(function()
					task.wait(1)
					thehum.PlatformStand = false
				end)
			end
		end
	end,
	["rightkicking"] = function(part, thehum) --distance kick
		if thehum and thehum:IsA("Humanoid") then
			thehum.Health = thehum.Health - (DAMAGE*0.15) --100% base and + 15% more dmg
			playremovesound("hitflesh"..math.random(1,3), part, 1+math.random(-20,20)/70, 0)
			local vel = Instance.new("BodyVelocity", part) --problem officer?
			vel.MaxForce = Vector3.new(1/0,1/0,1/0)
			vel.Velocity = charroot.CFrame.LookVector * 20
			debris:AddItem(vel, 0.2)
			if not thehum.PlatformStand and thehum:GetState() ~= Enum.HumanoidStateType.FallingDown then
				part.CFrame = CFrame.lookAt(part.Position, Vector3.new(charhead.Position.x,part.Position.y,charhead.Position.z))
				thehum.PlatformStand = true
				task.spawn(function()
					task.wait(0.5)
					thehum.PlatformStand = false
				end)
			end
		end
	end,
}
local animsequence = {
	["equip"] = function(savedtick, speed)
		playsound("equip", 2+math.random(-15,5)/50, 0.05)
		for i = 1,2 do
			if savedtick == currentanimtick then
				pose("equip", i, speed, savedtick)
				task.wait(0.1*speed)
			end
		end
		if equipped() then
			playsound("knuckles", 1.25+math.random(-15,5)/50)
		end
		task.wait(0.05*speed)
		task.spawn(function()
			task.wait(0.3*speed)
			if savedtick == currentanimtick then
				playsound("knuckles", 1.25+math.random(-15,5)/50)
			end
		end)
		for i = 3,5 do
			if savedtick == currentanimtick then
				pose("equip", i, speed, savedtick)
				task.wait(0.15*speed)
			end
		end
		task.wait(0.1*speed)
		pose("idle", 1, speed, savedtick)
	end,
	["idle"] = function(savedtick, speed)
		pose("idle", 1, speed, savedtick)
	end,
	["swing1"] = function(savedtick, speed)
		for i = 1,3 do
			if savedtick == currentanimtick then
				pose("swing1", i, speed, savedtick)
				task.wait(0.2*speed)
			end
		end
		for i = 4,5 do
			if savedtick == currentanimtick then
				pose("swing1", i, speed, savedtick)
				task.wait(0.4*speed)
			end
		end
		if savedtick == currentanimtick then
			pose("idle", 1, speed*4, savedtick)
		end
	end,
	["swing2"] = function(savedtick, speed)
		for i = 1,3 do
			if savedtick == currentanimtick then
				pose("swing2", i, speed, savedtick)
				task.wait(0.2*speed)
			end
		end
		for i = 4,5 do
			if savedtick == currentanimtick then
				pose("swing2", i, speed, savedtick)
				task.wait(0.4*speed)
			end
		end
		if savedtick == currentanimtick then
			pose("idle", 1, speed*4, savedtick)
		end
	end,
	["swing3"] = function(savedtick, speed)
		for i = 1,3 do
			if savedtick == currentanimtick then
				pose("swing3", i, speed, savedtick)
				task.wait(0.2*speed)
			end
		end
		for i = 4,5 do
			if savedtick == currentanimtick then
				pose("swing3", i, speed, savedtick)
				task.wait(0.4*speed)
			end
		end
		if savedtick == currentanimtick then
			pose("idle", 1, speed*4, savedtick)
		end
	end,
	["swing4"] = function(savedtick, speed)
		for i = 1,3 do
			if savedtick == currentanimtick then
				pose("swing4", i, speed, savedtick)
				task.wait(0.2*speed)
			end
		end
		for i = 4,5 do
			if savedtick == currentanimtick then
				pose("swing4", i, speed, savedtick)
				task.wait(0.4*speed)
			end
		end
		if savedtick == currentanimtick then
			pose("idle", 1, speed*4, savedtick)
		end
	end,
	["block"] = function(savedtick, speed)
		playsound("equip", 2+math.random(-5,20)/50, 0)
		pose("block", 1, speed, savedtick)
	end,
	["parrydodge"] = function(savedtick, speed)
		playremovesound("block"..math.random(1,3), charhead, 1+math.random(-10,10)/30, 0)
		pose("block", 2, speed, savedtick)
	end,
	["kick"] = function(savedtick, speed)
		pose("kick", 1, speed*2.5, savedtick)
		task.wait(0.2*(speed*2.5))
		for i = 2,3 do
			if savedtick == currentanimtick then
				pose("kick", i, speed*0.75, savedtick)
				task.wait(0.15*speed)
			end
		end
		pose("kick", 4, speed*3.5, savedtick)
		task.wait(0.2*(speed*3.5))
	end,
	["rlegkick"] = function(savedtick, speed)
		pose("rlegkick", 1, speed*1.5, savedtick)
		task.wait(0.2*(speed*1.5))
		for i = 2,3 do
			if savedtick == currentanimtick then
				pose("rlegkick", i, speed*1.2, savedtick)
				task.wait(0.2*(speed*1.2))
			end
		end
	end,
	["attemptcounter"] = function(savedtick, speed)
		pose("attemptcounter", 1, speed, savedtick)
		playsound("equip", 3+math.random(-5,20)/50, 0)
	end,
	["counter"] = function(savedtick, speed)
		playremovesound("block"..math.random(1,3), charhead, 1+math.random(-10,10)/30, 0)
		pose("counter", 1, speed, savedtick)
		task.wait(speed*0.15)
		pose("counter", 2, speed, savedtick)
		task.wait(speed*0.2)
		pose("counter", 3, speed, savedtick)
		task.wait(speed*0.15)
		pose("counter", 4, speed, savedtick)
		task.wait(speed*0.2)
		playremovesound("swing2", charhead, 0.7+math.random(-5,20)/50, 0)
		playremovesound("fabric", charhead, 1+math.random(-5,20)/50, 0)
		pose("counter", 5, speed, savedtick)
		task.wait(speed*0.1)
		playremovesound("grapple"..math.random(1,2), charhead, 0.8+math.random(-10,10)/30, 0)
		task.spawn(function()
			task.wait(0.1)
			playremovesound("fabric", charhead, 0.7+math.random(-5,20)/50, 0)
			playremovesound("swing1", charhead, 0.7+math.random(-5,20)/50, 0)
		end)
		pose("counter", 6, speed, savedtick)
		for i = 7,9 do
			task.wait(speed*0.125)
			pose("counter", i, speed, savedtick)
		end
		task.wait(speed*0.15)
		currentwelds["rootweld"].C0 = currentwelds["rootweld"].C0 * CFrame.Angles(0,math.pi,0)
		charroot.CFrame = charroot.CFrame * CFrame.Angles(0,math.pi,0)
		pose("idle", 1, speed, savedtick)
		pose("resetkicklegs", 1, speed, savedtick)
	end,
}
local remotekeybehavior = {
	["f"] = function()
		if toolstate == "idle" or toolstate == "swing" then
			toolstate = "block"
			currentanimtick = tick()
			remote:FireClient(owner, "suddenraystop")
			local nailedtheblock = false
			local takenaway = charhum.WalkSpeed*blockwsPERCENTAGE
			local btick = currentanimtick
			local parryfunc
			local lasthealth = charhum.Health
			charhum.WalkSpeed = charhum.WalkSpeed - takenaway
			parryfunc = charhum:GetPropertyChangedSignal("Health"):Connect(function()
				if charhum.Health < lasthealth then
					--print('regen')
					charhum.Health = lasthealth
					if not nailedtheblock then
						--print('parry anim')
						toolstate = "idle"
						nailedtheblock = true
						charhum.WalkSpeed = charhum.WalkSpeed + takenaway
						task.spawn(function() --anti platform stand
							for i = 1,10 do
								charhum.PlatformStand = false --.be/b_C7E2xMa7I?t=250
								task.wait()
							end
						end)
						remote:FireClient(owner, "stopbar", "block")
						animsequence["parrydodge"](btick, 0.6)
						task.wait(0.08)
						remote:FireClient(owner, "changestate", Enum.HumanoidStateType.GettingUp) --WAKE UP!!
						if toolstate == "idle" then
							animsequence["idle"](btick, 1)
							remote:FireClient(owner, "isholding")
						end
						task.spawn(function() --immunity frames
							task.wait(0.4)
							parryfunc:Disconnect()
						end)
					end
				end
				--print(charhum.Health)
				lasthealth = charhum.Health
				--print('hp updated',lasthealth)
			end)
			task.spawn(function()
				animsequence["block"](btick, 1)
			end)
			remote:FireClient(owner, "bar", "block", 1+0.5, false)
			task.wait(1)
			if not nailedtheblock then
				parryfunc:Disconnect()
				animsequence["idle"](btick, 1)
				task.wait(0.5)
				charhum.WalkSpeed = charhum.WalkSpeed + takenaway
				toolstate = "idle"
				remote:FireClient(owner, "isholding")
			end
		end
	end,
	["r"] = function()
		if (toolstate == "idle" or toolstate == "swing") and cancounter then
			toolstate = "counter"
			cancounter = false
			remote:FireClient(owner, "suddenraystop")
			currentanimtick = tick()
			local livingBEINGS = {}
			local btick = currentanimtick
			local takenaway = charhum.WalkSpeed*counterwsPERCENTAGE
			local counterfunc
			local nailedthecounter = false
			local lasthealth = charhum.Health
			charhum.WalkSpeed = charhum.WalkSpeed - takenaway
			for i,v in pairs(workspace:GetDescendants()) do
				if v:IsA("Model") and v ~= character and v:FindFirstChildOfClass("Humanoid") and v:FindFirstChildOfClass("Humanoid").Health > 0 and v:FindFirstChild("Head") then
					livingBEINGS[v] = {v:FindFirstChildOfClass("Humanoid"), v.Head}
				end
			end
			counterfunc = charhum:GetPropertyChangedSignal("Health"):Connect(function()
				if charhum.Health < lasthealth then
					--print('regen')
					local closestdist = counterdetectionMAG
					local closestperson
					for i,v in pairs(livingBEINGS) do
						local DISTANCE = (v[2].Position - charhead.Position).Magnitude
						if DISTANCE < closestdist then
							closestperson = i; closestdist = DISTANCE
						end
					end
					if closestperson then
						charhum.Health = lasthealth
						if not nailedthecounter and closestperson:FindFirstChild("Torso") then
							--print('parry anim')
							nailedthecounter = true
							remote:FireClient(owner, "changestate", Enum.HumanoidStateType.GettingUp) --WAKE UP now
							remote:FireClient(owner, "trackbool", false, false)
							remote:FireClient(owner, "stopbar", "counter")
							task.spawn(function() --anti platform stand
								for i = 1,10 do
									charhum.PlatformStand = false
									task.wait()
								end
							end)
							if not closestperson:FindFirstChild("HumanoidRootPart") then
								faketorso(closestperson)
							end
							local preal = players:GetPlayerFromCharacter(closestperson)
							local proot = closestperson:FindFirstChild("HumanoidRootPart") or closestperson.Torso
							local ptorso = closestperson.Torso
							local phum = closestperson:FindFirstChildOfClass("Humanoid")
							local lastrot = phum.AutoRotate
							local savedtorsotransparency = closestperson.Torso.Transparency
							local savedshirtid
							local takentools = false
							local theitems = {}
							local temp
							phum.AutoRotate = false
							pcall(function()
								temp = Instance.new("Folder", preal:FindFirstChildOfClass("Backpack"))
								if preal.Character:FindFirstChildOfClass("Tool") then
									preal.Character:FindFirstChildOfClass("Tool").Parent = preal:FindFirstChildOfClass("Backpack")
								end
								for i,v in pairs(preal:FindFirstChildOfClass("Backpack"):GetChildren()) do
									if v:IsA("Tool") or v:IsA("HopperBin") then
										table.insert(theitems, v)
										v.Parent = temp
									end
								end
								takentools = true
							end)
							if proot == ptorso then
								ptorso = closestperson.torsoholder.Torso
							end
							unweldasdescendantof(closestperson) --miner spelling mistake
							weld(chartorso, character["Left Leg"], CFrame.new(-0.5,-2,0), "llegweld")
							weld(chartorso, character["Right Leg"], CFrame.new(0.5,-2,0), "rlegweld")
							weld(ptorso, closestperson.Head, CFrame.new(0,1.5,0), "headweldGUY")
							weld(ptorso, closestperson["Right Arm"], CFrame.new(1.5,0.5,0) * CFrame.Angles(math.pi/2,0,0) * CFrame.new(0,-0.5,0), "rarmweldGUY")
							weld(ptorso, closestperson["Left Arm"], CFrame.new(-1.5,0.5,0) * CFrame.Angles(math.pi/2,0,0) * CFrame.new(0,-0.5,0), "larmweldGUY")
							weld(ptorso, closestperson["Right Leg"], CFrame.new(0.5,-2,0), "rlegweldGUY")
							weld(ptorso, closestperson["Left Leg"], CFrame.new(-0.5,-2,0), "llegweldGUY")
							if closestperson:FindFirstChild("HumanoidRootPart") then
								weld(closestperson.HumanoidRootPart, ptorso, CFrame.new(), "rootweldGUY")
							else
								closestperson.Torso.Transparency = 1
								if closestperson.Torso:FindFirstChildOfClass("ShirtGraphic") then
									savedshirtid = closestperson.Torso:FindFirstChildOfClass("ShirtGraphic").Graphic
									closestperson.Torso:FindFirstChildOfClass("ShirtGraphic").Graphic = "XD"
								end
								weld(closestperson.Torso, closestperson.torsoholder.Torso, CFrame.new(), "rootweldGUY")
							end
							task.spawn(function()
								for i = 1,10 do
									charroot.CFrame = CFrame.lookAt(charroot.Position, Vector3.new(proot.Position.x, charroot.Position.y, proot.Position.z))
									proot.CFrame = charroot.CFrame * CFrame.new(0,0,-3.25) * CFrame.Angles(0,math.pi,0)
									task.wait()
								end
							end)
							local bodyp = Instance.new("BodyPosition", proot)
							bodyp.MaxForce = Vector3.new(1/0,0,1/0)
							bodyp.Position = proot.Position
							bodyp.Name = "parrid"
							local bodyg = Instance.new("BodyGyro", proot)
							bodyg.MaxTorque = Vector3.new(1/0,1/0,1/0)
							bodyg.CFrame = proot.CFrame
							local bodyg1 = Instance.new("BodyGyro", charroot)
							bodyg1.MaxTorque = Vector3.new(1/0,1/0,1/0)
							bodyg1.CFrame = charroot.CFrame
							local bodyp1 = Instance.new("BodyPosition", charroot)
							bodyp1.MaxForce = Vector3.new(1/0,0,1/0)
							bodyp1.Position = charroot.Position
							task.spawn(function()
								task.wait(0.2)
								charroot.Anchored = true
								proot.Anchored = true
							end)
							animsequence["counter"](btick, 0.9)
							local savepos = ptorso.CFrame
							phum.RequiresNeck = false
							unweldasdescendantof(closestperson)
							if closestperson:FindFirstChild("torsoholder") then
								closestperson.torsoholder:Destroy()
								if closestperson.Torso:FindFirstChildOfClass("ShirtGraphic") then
									closestperson.Torso:FindFirstChildOfClass("ShirtGraphic").Graphic = savedshirtid
								end
							end
							closestperson.Torso.Transparency = savedtorsotransparency
							proot.CFrame = savepos * CFrame.Angles(-1,0,0)
							proot.Anchored = false
							phum.AutoRotate = lastrot
							bodyp:Destroy()
							bodyg:Destroy()
							proot.Velocity = Vector3.new()
							proot.AssemblyAngularVelocity = Vector3.new()
							local THROWHIM = Instance.new("BodyVelocity", proot)
							THROWHIM.MaxForce = Vector3.new(1/0,500,1/0)
							THROWHIM.Velocity = charroot.CFrame.LookVector * 20
							local SPINHIM = Instance.new("BodyAngularVelocity", proot)
							SPINHIM.MaxTorque = Vector3.new(0,1/0,0)
							SPINHIM.AngularVelocity = Vector3.new(0,-15,0)
							debris:AddItem(THROWHIM, 0.4)
							debris:AddItem(SPINHIM, 0.5)
							task.spawn(function()
								task.wait(0.25)
								pcall(function()
									for i,v in pairs(theitems) do
										v.Parent = preal:FindFirstChildOfClass("Backpack")
									end
									temp:Destroy()
								end)
								phum.RequiresNeck = true
								phum.Health = phum.Health - counterDAMAGE
								removeweld("rlegweld")
								removeweld("llegweld")
								bodyp1:Destroy()
								bodyg1:Destroy()
								charroot.Anchored = false
								local pleasework = Instance.new("BodyVelocity", charroot)
								local pleaseworkk = Instance.new("BodyAngularVelocity", charroot)
								pleasework.MaxForce = Vector3.new(1/0,1/0,1/0)
								pleaseworkk.MaxTorque = Vector3.new(1/0,1/0,1/0)
								pleasework.Velocity = Vector3.new()
								pleaseworkk.AngularVelocity = Vector3.new()
								task.wait(0.05)
								pleasework:Destroy()
								pleaseworkk:Destroy()
								charhum.WalkSpeed = charhum.WalkSpeed + takenaway
								toolstate = "idle"
								remote:FireClient(owner, "trackbool", true, true)
								remote:FireClient(owner, "isholding")
								remote:FireClient(owner, "changestate", Enum.HumanoidStateType.GettingUp) --again incase humanoid shenanigans
								cancounter = true
								task.wait(0.4) --immunity frames
								if takentools then
									phum.WalkSpeed = 16
									phum.JumpPower = 50
								end
								counterfunc:Disconnect()
							end)
						end
					end
				end
				--print(charhum.Health)
				lasthealth = charhum.Health
				--print('hp updated',lasthealth)
			end)
			task.spawn(function()
				animsequence["attemptcounter"](btick, 0.9)
			end)
			remote:FireClient(owner, "bar", "counter", 0.7+counterFAILEDcooldown, false)
			task.wait(0.7)
			if not nailedthecounter then
				counterfunc:Disconnect()
				animsequence["idle"](btick, 1)
				task.spawn(function()
					task.wait(counterFAILEDcooldown)
					cancounter = true
				end)
				task.wait(1.25)
				charhum.WalkSpeed = charhum.WalkSpeed + takenaway
				toolstate = "idle"
				remote:FireClient(owner, "isholding")
			end
		end
	end,
	["e"] = function() --RIGHT
		if (toolstate == "idle" or toolstate == "swing") and cankick then
			toolstate = "rightkicking"
			cankick = false
			table.clear(peoplehit)
			--remote:FireClient(owner, "suddenraystop")
			remote:FireClient(owner, "bar", "kick", (0.815*0.4)+0.25+rightkickcooldown, false)
			remote:FireClient(owner, "startray", "Right Leg", 0.45, true)
			playremovesound("swing2", charhead, 0.7+math.random(-5,20)/50, 0)
			playremovesound("fabric", charhead, 1+math.random(-5,20)/50, 0)
			currentanimtick = tick()
			local btick = currentanimtick
			local takenaway = charhum.WalkSpeed*kickwsPERCENTAGE
			charhum.WalkSpeed = charhum.WalkSpeed - takenaway
			weld(chartorso, character["Left Leg"], CFrame.new(-0.5,-2,0), "llegweld")
			weld(chartorso, character["Right Leg"], CFrame.new(0.5,-2,0), "rlegweld")
			currenttrails["Right Leg"].Enabled = true
			animsequence["rlegkick"](btick, 0.4)
			pose("idle", 1, 1.5, btick)
			pose("resetkicklegs", 1, 1.5, btick)
			task.spawn(function()
				charhum.WalkSpeed = charhum.WalkSpeed + takenaway
				task.wait(0.25)
				removeweld("llegweld")
				removeweld("rlegweld")
				task.wait(rightkickcooldown)
				cankick = true
			end)
			currenttrails["Right Leg"].Enabled = false
			task.wait(0.25)
			toolstate = "idle"
			remote:FireClient(owner, "isholding")
		end
	end,
	["q"] = function()
		if (toolstate == "idle" or toolstate == "swing") and cankick then
			toolstate = "leftkicking"
			cankick = false
			table.clear(peoplehit)
			--remote:FireClient(owner, "suddenraystop")
			remote:FireClient(owner, "bar", "kick", (1.55*0.3)+0.75+leftkickcooldown, false)
			remote:FireClient(owner, "startray", "Left Leg", 0.5, true)
			playremovesound("swing2", charhead, 0.7+math.random(-5,20)/50, 0)
			playremovesound("fabric", charhead, 1+math.random(-5,20)/50, 0)
			currentanimtick = tick()
			local btick = currentanimtick
			local takenaway = charhum.WalkSpeed*kickwsPERCENTAGE
			charhum.WalkSpeed = charhum.WalkSpeed - takenaway
			weld(chartorso, character["Left Leg"], CFrame.new(-0.5,-2,0), "llegweld")
			weld(chartorso, character["Right Leg"], CFrame.new(0.5,-2,0), "rlegweld")
			currenttrails["Left Leg"].Enabled = true
			animsequence["kick"](btick, 0.3)
			pose("idle", 1, 1.3, btick)
			pose("resetkicklegs", 1, 1.5, btick)
			task.spawn(function()
				task.wait(0.5)
				charhum.WalkSpeed = charhum.WalkSpeed + takenaway
				task.wait(0.25)
				removeweld("llegweld")
				removeweld("rlegweld")
				task.wait(leftkickcooldown)
				cankick = true
			end)
			currenttrails["Left Leg"].Enabled = false
			task.wait(0.3)
			toolstate = "idle"
			remote:FireClient(owner, "isholding")
		end
	end,
}
local remotebehavior = {
	["lmb"] = function(pos)
		if equipped() and toolstate == "idle" then
			toolstate = "swing"
			currentanimtick = tick()
			local btick = currentanimtick
			local beqtick = currentequiptick
			remote:FireClient(owner, "bar", "fist", (1.45*swingdatas[swinganimation][2])+0.05, false)
			playremovesound("swing2", charhead, 0.7+math.random(-5,20)/50, 0)
			playremovesound("fabric", charhead, 1+math.random(-5,20)/50, 0)
			table.clear(peoplehit)
			task.spawn(function()
				task.wait(0.07) --dont wanna scan during the windup anim
				if toolstate == "swing" and equipped() then
					remote:FireClient(owner, "startray", swingdatas[swinganimation][1], 1.35*swingdatas[swinganimation][2], true)
				end
			end)
			currenttrails[swingdatas[swinganimation][1]].Enabled = true
			animsequence["swing"..swinganimation](btick, swingdatas[swinganimation][2])
			currenttrails[swingdatas[swinganimation][1]].Enabled = false
			swinganimation = swinganimation + 1
			if swinganimation > #swingdatas then
				swinganimation = 1
			end
			task.wait(0.05)
			--the end
			if (currentanimtick == btick or currentequiptick ~= beqtick) and not table.find(illegalstates, toolstate) then
				toolstate = "idle"
				remote:FireClient(owner, "isholding")
			end
		end
	end,
	["part"] = function(thepart, thepos, thenormal)
		--print("hit attempt")
		if equipped() and not table.find(illegalstates, toolstate) and thepart then
			local foundhim = parentfindfirstchildofclass("Humanoid", thepart) or thepart
			if not peoplehit[foundhim] then
				peoplehit[foundhim] = thepart
				local wavecolor = thepart.Color
				local usedhand = character[swingdatas[swinganimation][1]]
				if foundhim:IsA("Humanoid") then
					if (thepart.Position - charroot.Position).Magnitude <= 8 then
						wavecolor = Color3.new(1,1,1)
						if foundhim.Health <= 0 then --just remove my name and random if you want to hear the sound\
							if math.random(1,50) == 1 and owner.Name == "Rufus14" then
								playremovesound("AUGHHHH", thepart, 1, 0)
							end
							foundhim.BreakJointsOnDeath = false
						end
						foundhim.Health = foundhim.Health - DAMAGE
						playremovesound("punch"..math.random(1,5), thepart, 1+math.random(-20,20)/70, 0)
						if notable_nerve_response[thepart.Name] then
							wavecolor = Color3.new(1,0,0)
							notable_nerve_response[thepart.Name](thepart, foundhim)
						end
						if notable_state_response[toolstate] then
							notable_state_response[toolstate](thepart, foundhim)
						end
					end
				else
					playremovesound("concrete", usedhand, 1+math.random(-20,20)/70, 0)
				end
				local handtopos = (usedhand.Position - thepos)
				local clampmag = math.clamp(handtopos.Magnitude,0,4)
				local shockwave = makepart(
					workspace,
					Vector3.new(0.5,0.5,0.5),
					CFrame.new(usedhand.Position-(handtopos.Unit*clampmag), (usedhand.Position-(handtopos.Unit*clampmag)) + thenormal) * CFrame.Angles(-math.pi/2,-math.pi/2,0), --imagine my shock
					true,
					false, 
					"walmart tier hit effect"
				)
				shockwave.CastShadow = false
				shockwave.CanTouch = false
				shockwave.CanQuery = false --false ladder
				shockwave.Color = wavecolor
				local shockmesh = Instance.new("SpecialMesh", shockwave)
				shockmesh.MeshId = "http://www.roblox.com/asset/?id=20329976"
				shockmesh.Scale = Vector3.new(0.5,1.5,0.5)
				debris:AddItem(shockwave, 0.2)
				tween(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out, 0, shockwave, {Transparency = 1, CFrame = shockwave.CFrame * CFrame.Angles(0,1,0) * CFrame.new(0,0.5,0)})
				tween(0.1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out, 0, shockmesh, {Scale = Vector3.new(1.5,0.15,1.5)})
			end
		end
	end,
	["keypress"] = function(thekey)
		if remotekeybehavior[thekey] and equipped() then
			remotekeybehavior[thekey]()
		end
	end,
}
motor = function(part0, part1, c0, name)
	local m = Instance.new("Motor6D")
	m.Part0 = part0
	m.Part1 = part1
	m.C0 = c0
	m.Name = name
	m.Parent = part0
	currentwelds[m.Name] = m
end
parentfindfirstchildofclass = function(cname, search)
	local par = search
	local foundinstance
	while par ~= workspace and not foundinstance do
		foundinstance = par:FindFirstChildOfClass(cname)
		par = par.Parent
	end
	return foundinstance
end
for i,v in pairs(sfxdata) do --dont look at this in particular
	if v[5] then
		makeplayonremovesound(v[1],v[2],v[4],v[3],1,i)
	else
		makesound(v[1],nil,v[2],v[4],v[3],1,i)
	end
end
remote.OnServerEvent:Connect(function(WHO, WHAT, ...)
	if WHO ~= owner then 
		if WHO.Character then 
			WHO.Character:BreakJoints() 
		end 
		return nil
	end
	if remotebehavior[WHAT] then
		remotebehavior[WHAT](...)
		--print("check")
	end
end)
tool.Equipped:Connect(function()
	currentanimtick = tick()
	currentequiptick = currentanimtick
	local backuptick = currentanimtick
	owner = players:GetPlayerFromCharacter(tool.Parent)
	character = owner.Character
	charhead = character.Head
	charhum = character:FindFirstChildOfClass("Humanoid")
	charroot = character.HumanoidRootPart
	chartorso = character.Torso
	table.insert(ignoretable, character)
	lookrootpart = Instance.new("Part")
	lookrootpart.Size = Vector3.new(0.5,0.5,0.5)
	lookrootpart.CanCollide = false
	lookrootpart.CanTouch = false
	lookrootpart.CanQuery = false
	lookrootpart.Name = "lrp"
	lookrootpart.Transparency = 1
	lookrootpart.Parent = character.Head
	lookrootpart:BreakJoints()
	--
	aimpart = makepart(
		character,
		Vector3.new(0.5,0.5,0.5),
		charhead.CFrame * CFrame.new(0,-0.5,1.5),
		false,
		false,
		"aimpartfist"
	)
	aimpart.Transparency = 1
	aimpart.Shape = Enum.PartType.Ball
	aimpart.Material = Enum.Material.Neon
	aimpart.CanQuery = false
	aimpart.CanTouch = false
	aimpart:SetNetworkOwner(owner) --GHOST ! ! ! GHOST ! ! ! (busybox.flac)
	--
	weld(lookrootpart, character.Head, CFrame.new(0,1.5-(1+lookpartheight),0), "headtorootweld")
	weld(lookrootpart, character["Right Arm"], CFrame.new(1.5,-(1+lookpartheight),0), "rarmweld")
	weld(lookrootpart, character["Left Arm"], CFrame.new(-1.5,-(1+lookpartheight),0), "larmweld")
	weld(character.Torso, charroot, CFrame.new(), "rootweld")
	weld(character.Torso, lookrootpart, CFrame.new(0,1+lookpartheight,0), "lookrootweld")
	--
	if not character:FindFirstChild(attachmentname, true) then
		for i,v in pairs(character:GetDescendants()) do
			if attachmentdata[v.Name] then
				if currentattachs[v.Name] then
					continue
				end
				currentattachs[v.Name] = {}
				--print("added "..v.Name)
				local highestattachment
				local lowestattachment
				for q,w in pairs(attachmentdata[v.Name].placement) do
					for i = 1,attachmentdata[v.Name].repeatY do
						local at = Instance.new("Attachment", v)
						at.Visible = false
						at.Name = attachmentname
						at.Position = w + Vector3.new(0,attachmentdata[v.Name].depth*(i-1),0) --lua starting index at 1 GET IT?? Xd
						--table.insert(currentattachs[v.Name], at) --i dont think the attachments will be used directly on server so why bother wasting memory (this comment takes up approximately 214 bytes including the table.insert code)
						if q == 1 then --WHAT is this?/
							--print("hi")
							if i == 1 then
								--print("first")
								lowestattachment = at
							elseif i == attachmentdata[v.Name].repeatY then
								--print("no one caare")
								highestattachment = at
							end
						end	
					end
				end
				local trail = Instance.new("Trail", v)
				trail.Attachment0 = lowestattachment
				trail.Attachment1 = highestattachment
				trail.Lifetime = 0.1
				trail.Enabled = false
				trail.Transparency = NumberSequence.new(0.7,1)
				trail.FaceCamera = true
				trail.WidthScale = NumberSequence.new(1,0)
				currenttrails[v.Name] = trail
			end
		end
	end
	for i,v in pairs(currentsounds) do
		if not v.PlayOnRemove then
			v.Parent = charhead
		end
	end
	--
	runfunc = runservice.Stepped:Connect(function(_, delta)
		local probheadpos = charroot.Position + Vector3.new(0,1+lookpartheight,0)
		local theunit = (charhead.Position - aimpart.Position).unit
		local unitrel = charroot.CFrame:vectorToObjectSpace(theunit)
		local velrel = charroot.CFrame:vectorToObjectSpace(charroot.Velocity)
		local theXtilt = unitrel.x
		if velrel.x < -5 or velrel.x > 5 then
			theXtilt = 0
		end
		currentwelds["lookrootweld"].C0 = currentwelds["lookrootweld"].C0:lerp(CFrame.new(0,1+lookpartheight,0) * CFrame.Angles(-unitrel.y/1.5,theXtilt/1.5,0),delta*15)
	end)
	task.spawn(function()
		animsequence["equip"](currentanimtick, 1)
	end)
	remote:FireClient(owner, "init", attachmentname)
end)

tool.Unequipped:Connect(function()
	runfunc:Disconnect()
	unweldasdescendantof(character)
	for i,v in pairs(currentsounds) do
		if not v.PlayOnRemove then
			v.Parent = nil
		end
	end
	table.clear(currentwelds)
	table.remove(ignoretable, table.find(ignoretable, character))
	aimpart:Destroy()
	lookrootpart:Destroy()
end)
