multiwave
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;
using Voxels.SetRules;

namespace Voxels
{
	public class MultiWave
	{
		private MultiWave()
		{
		}

		/// <summary>
		/// Standard constructor. If <paramref name="size"/> is Vector3Int.zero, an optimal size will be
		/// generated from the given seeds, respecting the 512‑tile volume limit.
		/// </summary>
		public MultiWave(string name, Vector3Int size, int seed, int seed2, float minimumBeach, int coinTarget, List<string> tilesets)
		{
			this.name = name;
			this.seed = seed;
			this.seed2 = seed2;
			this.minimumBeach = minimumBeach;
			this.coinTarget = coinTarget;
			this.tilesetKeys = tilesets;

			// Auto‑size if no valid size was supplied
			if (size == Vector3Int.zero)
			{
				size = GenerateSizeFromSeeds(seed, seed2);
			}
			else
			{
				// Enforce hard limits: max width 16, max height 8, max volume 512
				int maxAllowedWidth = Mathf.Min(16, Mathf.FloorToInt(Mathf.Sqrt(512f / Mathf.Max(1, size.y))));
				size.x = Mathf.Clamp(size.x, 1, maxAllowedWidth);
				size.z = size.x; // keep square
				size.y = Mathf.Clamp(size.y, 1, 8);
			}
			this.size = size;
			this.heightTier = Mathf.Clamp(size.y, 1, 8);
		}

		private PlacementManager placementManager
		{
			get { return Singleton<PlacementManager>.instance; }
		}

		public Vector3 moduleOffset
		{
			get { return new Vector3((float)(-(float)(this.size.x - 1)) / 2f, 0f, (float)(-(float)(this.size.z - 1)) / 2f); }
		}

		public Matrix4x4 moduleMatrix
		{
			get { return this.moduleOffset.GetMoveMatrix(); }
		}

		public int remainingDominos
		{
			get { return this.allDominos.Count - this.removedDominos - this.placedDominos; }
		}

		public MultiWave.Phase phase { get; private set; }

		public void Clear()
		{
			for (int i = 0; i < this.slots.Length; i++)
			{
				this.slots[i].Clear();
				this.slots[i] = null;
			}
			for (int j = 0; j < this.cornerSlots.Length; j++)
			{
				this.cornerSlots[j].Clear();
				this.cornerSlots[j] = null;
			}
			this.openSlots.Clear();
			this.guessQueue.Clear();
			foreach (Domino domino in this.allDominos)
			{
				domino.Clear();
			}
			this.allDominos.Clear();
		}

		public Slot GetSlot(Vector3Int pos)
		{
			return this.slots[ExtraMath.CoordinateToIndex(pos, this.size)];
		}

		public CornerSlot GetCorner(Vector3 pos)
		{
			return this.cornerSlots[ExtraMath.CoordinateToIndex(pos, this.size + Vector3Int.one)];
		}

		private void AddPlacement(Wrapper placementWrapper, Vector3Int offset, ref List<Domino> dominoList)
		{
			Placement placement = placementWrapper.placement;
			Vector3 point = placement.bounds.min + offset;
			Vector3 point2 = placement.bounds.max + offset;
			if (!this.bounds.Contains(point) || !this.bounds.Contains(point2))
			{
				return;
			}
			for (int i = 0; i < placementWrapper.allowPlacement.Count; i++)
			{
				if (!placementWrapper.allowPlacement[i].AllowPlacement(offset, placementWrapper.placement, this))
				{
					return;
				}
			}
			for (int j = 0; j < placement.claims.Count; j++)
			{
				Claim claim = placement.claims[j];
				Vector3Int a = claim.pos + offset;
				for (int k = 0; k < Constants.corners.Length; k++)
				{
					CornerSlot.Mode mode = this.GetCorner(a + Constants.cornersInt[k]).mode;
					if (mode != CornerSlot.Mode.any)
					{
						if (claim.cornersInside[k] != (mode == CornerSlot.Mode.inside))
						{
							return;
						}
					}
				}
				if (a.y == 0)
				{
					if (claim.edges[5].edges.Count > 0)
					{
						return;
					}
					for (int l = 0; l < 5; l++)
					{
						Vector3Int b = Constants.directions[l];
						Vector3Int v = a + b;
						if (!this.bounds.Contains(v))
						{
							if (!this.placementManager.lowEdge.Fits(claim.keys[l], l))
							{
								return;
							}
						}
					}
				}
				if (a.y > 0)
				{
					for (int m = 0; m < 6; m++)
					{
						Vector3Int v2 = a + Constants.directions[m];
						if (!this.bounds.Contains(v2))
						{
							if (claim.mode[m] != Claim.Mode.Outside)
							{
								return;
							}
						}
					}
				}
			}
			Domino domino = new Domino(placementWrapper, offset);
			for (int n = 0; n < placement.claims.Count; n++)
			{
				Claim claim2 = placement.claims[n];
				this.GetSlot(claim2.pos + offset).AddDomino(domino, claim2);
				for (int num = 0; num < Constants.corners.Length; num++)
				{
					CornerSlot corner = this.GetCorner(claim2.pos + offset + Constants.cornersInt[num]);
					corner.AddDomino(domino, num, claim2.cornersInside[num]);
				}
			}
			foreach (IOnDominoCreated onDominoCreated in placementWrapper.onDominoAdded)
			{
				onDominoCreated.OnDominoAdded(domino, this);
			}
			dominoList.Add(domino);
		}

		public IEnumerable Setup()
		{
			this.size.y = this.heightTier;
			this.phase = MultiWave.Phase.Setup;
			this.openBounds = default(Bounds);
			this.navigableBounds = default(Bounds);
			Vector3Int marcherSize = this.size - Vector3Int.one * 2;
			marcherSize.y = 0;
			this.marcher = new StraightMarcher(marcherSize);
			this.marcher.Bend(-Mathf.Tan(0.5235988f));
			this.slots = new Slot[this.size.x * this.size.y * this.size.z];
			this.bounds = default(Bounds);
			this.bounds.Encapsulate(this.size - Vector3.one);
			this.bounds.extents = this.bounds.extents + Vector3.one / 2f;
			this.cornerSlots = new CornerSlot[(this.size.x + 1) * (this.size.y + 1) * (this.size.z + 1)];
			this.resolvedSlotsPerHeight = new int[this.size.y];
			this.savedResolvedSlotsPerHeight = new int[this.size.y];
			this.goldPerHeight = new int[this.size.y];
			for (int i = 0; i < this.cornerSlots.Length; i++)
			{
				Vector3Int pos = ExtraMath.IndexToCoordinate(i, this.size + Vector3Int.one);
				CornerSlot cornerSlot = new CornerSlot(pos, this.marcher.rays.Count);
				this.cornerSlots[i] = cornerSlot;
				if (pos.y == 0)
				{
					cornerSlot.mode = CornerSlot.Mode.any;
				}
				else if (!this.bounds.Contains(pos - Vector3.one / 2f))
				{
					cornerSlot.mode = CornerSlot.Mode.outside;
				}
				else
				{
					cornerSlot.mode = CornerSlot.Mode.any;
				}
				yield return null;
			}
			for (int j = 0; j < this.slots.Length; j++)
			{
				Slot slot = new Slot(ExtraMath.IndexToCoordinate(j, this.size), this.placementManager.maxKeyCount);
				this.slots[j] = slot;
				yield return null;
			}
			ModuleSet[] allSets = this.placementManager.moduleSets;
			List<ModuleSet> excludedSets = (from x in allSets where !x.enabledByDefault && !this.tilesetKeys.Contains(x.cachedName) select x).ToList<ModuleSet>();
			float bestHouseSetScore = float.MinValue;
			foreach (ModuleSet moduleSet in allSets)
			{
				if (!excludedSets.Contains(moduleSet))
				{
					HouseSet houseSet = null;
					foreach (SetRule setRule in moduleSet.rules)
					{
						if (setRule is HouseSet)
						{
							houseSet = (setRule as HouseSet);
							break;
						}
					}
					if (houseSet)
					{
						float houseSetScore = houseSet.houseSetScore;
						if (houseSetScore >= bestHouseSetScore)
						{
							bestHouseSetScore = houseSetScore;
							this.bestHouseSet = moduleSet;
						}
					}
				}
			}
			List<Wrapper> wrappers = new List<Wrapper>((int)((float)this.placementManager.all.Count * 0.7f));
			for (int num3 = 0; num3 < this.placementManager.all.Count; num3++)
			{
				Placement placement = this.placementManager.all[num3];
				if (!excludedSets.Any((ModuleSet x) => x.ContainsModule(placement.firstModule)))
				{
					Wrapper item = new Wrapper(placement);
					wrappers.Add(item);
				}
			}
			List<Wrapper> setWrappers = new List<Wrapper>();
			for (int num4 = 0; num4 < this.placementManager.moduleSets.Length; num4++)
			{
				ModuleSet moduleSet2 = this.placementManager.moduleSets[num4];
				if (!excludedSets.Contains(moduleSet2))
				{
					for (int num5 = 0; num5 < wrappers.Count; num5++)
					{
						Wrapper wrapper2 = wrappers[num5];
						if (moduleSet2.ContainsModule(wrapper2.placement.firstModule))
						{
							setWrappers.Add(wrapper2);
						}
					}
					for (int num6 = 0; num6 < moduleSet2.rules.Length; num6++)
					{
						moduleSet2.rules[num6].GetRules(this, setWrappers);
					}
					setWrappers.Clear();
				}
			}
			List<Domino> allInitalDominos = new List<Domino>(this.slots.Length * wrappers.Count);
			this.allDominos = new List<Domino>(this.slots.Length * wrappers.Count);
			this.guessQueue = new List<Domino>(this.allDominos.Count);
			this.openSlots = new List<Slot>(this.slots.Length);
			for (int k = 0; k < wrappers.Count; k++)
			{
				Wrapper wrapper = wrappers[k];
				for (int l = 0; l < this.slots.Length; l++)
				{
					Slot slot2 = this.slots[l];
					this.AddPlacement(wrapper, slot2.pos, ref allInitalDominos);
					yield return null;
				}
			}
			this.phase = MultiWave.Phase.InitialContradictions;
			for (int m = 0; m < allInitalDominos.Count; m++)
			{
				Domino domino = allInitalDominos[m];
				if (!this.DominoFits(domino))
				{
					this.RemoveDomino(domino);
					yield return null;
				}
			}
			for (int n = 0; n < allInitalDominos.Count; n++)
			{
				Domino domino2 = allInitalDominos[n];
				if (domino2.state == Domino.State.idle)
				{
					this.allDominos.Add(domino2);
				}
				yield return null;
			}
			this.removedDominos = 0;
			this.placedDominos = 0;
			this.phase = MultiWave.Phase.PlacingObvious;
			yield return null;
			for (int i2 = 0; i2 < this.slots.Length; i2++)
			{
				Slot slot3 = this.slots[i2];
				if (slot3.dominos.Count == 1 && slot3.dominos[0].state != Domino.State.done)
				{
					this.Place(slot3.dominos[0]);
				}
				yield return null;
			}
			yield return null;
			this.phase = MultiWave.Phase.SavingState;
			this.SaveState();
			yield break;
		}

		private void GenerateTopographyMap()
		{
			System.Random rng = new System.Random(this.seed ^ this.seed2 ^ 0x5A4B3C2D);
			float[] weights = new float[5];
			float total = 0f;
			for (int i = 0; i < 5; i++)
			{
				weights[i] = (float)rng.NextDouble();
				total += weights[i];
			}
			for (int i = 0; i < 5; i++)
			{
				weights[i] /= total;
			}
			int primaryProfile = 0;
			float maxWeight = weights[0];
			for (int i = 1; i < 5; i++)
			{
				if (weights[i] > maxWeight)
				{
					maxWeight = weights[i];
					primaryProfile = i;
				}
			}
			weights[primaryProfile] *= 1.5f;
			total = weights.Sum();
			for (int i = 0; i < 5; i++) weights[i] /= total;
			int width = this.size.x;
			int depth = this.size.z;
			this.topoMap = new float[width, depth];
			float perlinOffX = (float)rng.NextDouble() * 100f;
			float perlinOffZ = (float)rng.NextDouble() * 100f;
			float perlinScale = 0.12f + (float)rng.NextDouble() * 0.2f;
			int cellCount = Mathf.Clamp(width * depth / 8, 3, 20);
			Vector2[] cellPoints = new Vector2[cellCount];
			for (int i = 0; i < cellCount; i++)
				cellPoints[i] = new Vector2((float)rng.NextDouble() * width, (float)rng.NextDouble() * depth);
			for (int x = 0; x < width; x++)
			{
				for (int z = 0; z < depth; z++)
				{
					float val = 0f;
					float cx = (width - 1) * 0.5f;
					float cz = (depth - 1) * 0.5f;
					float dx = (x - cx) / Mathf.Max(cx, 1f);
					float dz = (z - cz) / Mathf.Max(cz, 1f);
					float dist = Mathf.Sqrt(dx * dx + dz * dz);
					float radial = 1f - Mathf.Clamp01(dist);
					float noiseVal = Mathf.PerlinNoise(perlinOffX + x * perlinScale, perlinOffZ + z * perlinScale);
					float minDist = float.MaxValue;
					for (int i = 0; i < cellPoints.Length; i++)
					{
						float d = (new Vector2(x, z) - cellPoints[i]).sqrMagnitude;
						if (d < minDist) minDist = d;
					}
					float cellVal = Mathf.Clamp01(1f - Mathf.Sqrt(minDist) / (width * 0.4f));
					float axisVal = Mathf.Max(1f - Mathf.Abs(dx), 1f - Mathf.Abs(dz));
					val += weights[0] * 0.5f;
					val += weights[1] * radial;
					val += weights[2] * noiseVal;
					val += weights[3] * cellVal;
					val += weights[4] * axisVal;
					this.topoMap[x, z] = Mathf.Clamp01(val);
				}
			}
			this.topographyBiasStrength = 0.4f + (float)rng.NextDouble() * 0.4f;
		}

		private void SaveState()
		{
			for (int i = 0; i < this.slots.Length; i++)
			{
				this.slots[i].SaveState();
			}
			for (int j = 0; j < this.allDominos.Count; j++)
			{
				this.allDominos[j].SaveState();
			}
			for (int k = 0; k < this.cornerSlots.Length; k++)
			{
				this.cornerSlots[k].SaveState();
			}
			for (int l = 0; l < this.size.y; l++)
			{
				this.savedResolvedSlotsPerHeight[l] = this.resolvedSlotsPerHeight[l];
			}
			this.removedDominosaved = this.removedDominos;
			this.placedDominosSaved = this.placedDominos;
			this.onSave(this);
		}

		private bool ForceVisibility(CornerSlot cornerSlot)
		{
			for (int i = 0; i < this.marcher.rays.Count; i++)
			{
				if (!cornerSlot.occludedAngles[i])
				{
					for (int j = 1; j < this.marcher.rays[i].pos.Length; j++)
					{
						Vector3Int v = cornerSlot.pos - this.marcher.rays[i].pos[j];
						if (!this.bounds.Contains(v))
						{
							break;
						}
						CornerSlot corner = this.GetCorner(v);
						if (corner.state == CornerSlot.State.Unclear)
						{
							if (corner.state == CornerSlot.State.Inside)
							{
								break;
							}
							for (int k = 0; k < 8; k++)
							{
								List<Domino> list = corner.dominos[k, 1];
								int l = 0;
								while (l < list.Count)
								{
									Domino domino = list[l];
									if (domino.state == Domino.State.idle)
									{
										if (!this.RemoveDomino(domino))
										{
											return false;
										}
									}
									else
									{
										l++;
									}
								}
							}
						}
					}
				}
			}
			return true;
		}

		private bool Occlude(CornerSlot cornerSlot)
		{
			for (int i = 0; i < this.marcher.rays.Count; i++)
			{
				for (int j = 1; j < this.marcher.rays[i].pos.Length; j++)
				{
					Vector3Int vector3Int = cornerSlot.pos + this.marcher.rays[i].pos[j];
					if (!this.bounds.Contains(vector3Int))
					{
						break;
					}
					if (vector3Int.y < 1)
					{
						break;
					}
					CornerSlot corner = this.GetCorner(vector3Int);
					if (corner.Occlude(i) && corner.visiblity < this.visibilityThreshold && this.GetCorner(vector3Int + Vector3Int.down).state == CornerSlot.State.Inside)
					{
						if (corner.state == CornerSlot.State.Outside)
						{
							if (!this.ForceVisibility(corner))
							{
								return false;
							}
						}
						else if (corner.state == CornerSlot.State.Unclear)
						{
							for (int k = 0; k < 8; k++)
							{
								List<Domino> list = corner.dominos[k, 0];
								int l = 0;
								while (l < list.Count)
								{
									Domino domino = list[l];
									if (domino.state == Domino.State.idle)
									{
										if (!this.RemoveDomino(domino))
										{
											return false;
										}
									}
									else
									{
										l++;
									}
								}
							}
						}
					}
				}
			}
			return true;
		}

		public bool RemoveDomino(Domino domino)
		{
			if (domino.state == Domino.State.idle)
			{
				Placement placement = domino.placement;
				Wrapper placementWrapper = domino.placementWrapper;
				domino.state = Domino.State.removing;
				this.removedDominos++;
				for (int i = 0; i < placement.claims.Count; i++)
				{
					Claim claim = placement.claims[i];
					Slot slot = this.GetSlot(domino.offset + claim.pos);
					if (slot.dominos.Remove(domino))
					{
						for (int j = 0; j < 6; j++)
						{
							int num = domino.GetClaim(slot.pos).keys[j];
							List<Domino> list = slot.keyCount[j, num];
							if (list.Remove(domino))
							{
								if (list.Count == 1)
								{
									Domino domino2 = list[0];
									Placement placement2 = domino2.placement;
									for (int k = 0; k < placementWrapper.onLast.Count; k++)
									{
										if (!placementWrapper.onLast[k].OnLast(domino2, this, j, claim, slot))
										{
											return false;
										}
									}
								}
								if (list.Count == 0)
								{
									Vector3Int vector3Int = slot.pos + Constants.directions[j];
									if (this.bounds.Contains(vector3Int))
									{
										Slot slot2 = this.GetSlot(vector3Int);
										List<Domino> list2 = slot2.keyCount[Constants.opposites[j], num];
										int l = 0;
										while (l < list2.Count)
										{
											Domino domino3 = list2[l];
											if (domino3.state == Domino.State.idle)
											{
												if (!this.RemoveDomino(domino3))
												{
													return false;
												}
											}
											else
											{
												l++;
											}
										}
									}
								}
							}
						}
						for (int m = 0; m < 8; m++)
						{
							CornerSlot corner = this.GetCorner(claim.pos + domino.offset + Constants.cornersInt[m]);
							int num2 = (!claim.cornersInside[m]) ? 0 : 1;
							List<Domino> list3 = corner.dominos[m, num2];
							if (list3.Remove(domino))
							{
								corner.stateCount[num2]--;
								if (list3.Count == 0)
								{
									for (int n = 0; n < 8; n++)
									{
										List<Domino> list4 = corner.dominos[n, num2];
										int num3 = 0;
										while (num3 < list4.Count)
										{
											Domino domino4 = list4[num3];
											if (domino4.state == Domino.State.idle)
											{
												if (!this.RemoveDomino(domino4))
												{
													return false;
												}
											}
											else
											{
												num3++;
											}
										}
									}
									if (corner.state == CornerSlot.State.Inside && !this.Occlude(corner))
									{
										return false;
									}
								}
							}
						}
						if (slot.dominos.Count == 1 && slot.dominos[0].state == Domino.State.idle)
						{
							if (!this.Place(slot.dominos[0]))
							{
								return false;
							}
						}
						else if (slot.dominos.Count == 0)
						{
							this.breakReason = "RemoveDomino()";
							this.broken = true;
							return false;
						}
					}
				}
				for (int num4 = 0; num4 < domino.placementWrapper.onRemoved.Count; num4++)
				{
					if (!domino.placementWrapper.onRemoved[num4].OnRemoved(domino, this))
					{
						return false;
					}
				}
			}
			return true;
		}

		public bool Place(Domino domino)
		{
			if (domino.state == Domino.State.idle)
			{
				domino.state = Domino.State.done;
				Placement placement = domino.placement;
				this.placedDominos++;
				this.beach += domino.placement.firstModule.beachLength;
				this.coinCount += domino.placement.firstModule.goldCount;
				this.goldPerHeight[domino.offset.y] += domino.placement.firstModule.goldCount;
				bool flag = false;
				if (domino.placement.navigable && this.navigating)
				{
					for (int i = 0; i < placement.claims.Count; i++)
					{
						Claim claim = placement.claims[i];
						if (claim.anyNavigable)
						{
							Slot slot = this.GetSlot(claim.pos + domino.offset);
							if (slot.navigability == Slot.Navigability.Open)
							{
								flag = true;
								break;
							}
							if (!flag)
							{
								for (int j = 0; j < 6; j++)
								{
									if (claim.navigable[j] && claim.mode[j] == Claim.Mode.Internal)
									{
										Slot slot2 = this.GetSlot(slot.pos + Constants.directions[j]);
										if (slot2.navigability != Slot.Navigability.Unclear)
										{
											flag = true;
											break;
										}
									}
								}
							}
							if (flag)
							{
								break;
							}
						}
					}
				}
				if (this.navigating && domino.placement.navigable)
				{
					this.navigableBounds.Encapsulate(domino.GetNavigableBounds());
					this.openBounds.Encapsulate(domino.GetNavigableBounds());
				}
				for (int k = 0; k < placement.claims.Count; k++)
				{
					Claim claim2 = placement.claims[k];
					Vector3Int pos = claim2.pos + domino.offset;
					Slot slot3 = this.GetSlot(pos);
					slot3.normal = claim2.normal;
					if (slot3.navigability == Slot.Navigability.Open && this.navigating)
					{
						slot3.navigability = Slot.Navigability.Navigable;
						this.openSlots.Remove(slot3);
						flag = true;
					}
					if (flag && claim2.anyNavigable && this.navigating)
					{
						slot3.navigability = Slot.Navigability.Navigable;
						for (int l = 0; l < 6; l++)
						{
							if (claim2.navigable[l] && claim2.mode[l] != Claim.Mode.Internal)
							{
								Slot slot4 = this.GetSlot(slot3.pos + Constants.directions[l]);
								if (!slot4.done && slot4.navigability == Slot.Navigability.Unclear)
								{
									slot4.navigability = Slot.Navigability.Open;
									this.openSlots.Add(slot4);
								}
							}
						}
					}
					if (slot3.domino != null)
					{
						this.breakReason = "Place() slot.domino != null";
						this.broken = true;
						return false;
					}
					slot3.domino = domino;
					this.resolvedSlotsPerHeight[pos.y]++;
					List<Domino> dominos = slot3.dominos;
					int m = 0;
					while (m < slot3.dominos.Count)
					{
						Domino domino2 = dominos[m];
						if (domino2 == domino || domino2.state == Domino.State.removing)
						{
							m++;
						}
						else
						{
							if (domino2.state == Domino.State.done)
							{
								this.breakReason = "otherDomino.state == Domino.State.done";
								this.broken = true;
								return false;
							}
							if (!this.RemoveDomino(domino2))
							{
								return false;
							}
						}
					}
				}
				if (!this.DominoFits(domino))
				{
					this.breakReason = "Place() !DominoFits(domino)";
					this.broken = true;
					return false;
				}
				for (int n = 0; n < domino.placementWrapper.onPlaced.Count; n++)
				{
					if (!domino.placementWrapper.onPlaced[n].OnPlaced(domino, this))
					{
						return false;
					}
				}
			}
			return true;
		}

		private bool DominoFits(Domino domino)
		{
			Placement placement = domino.placement;
			for (int i = 0; i < placement.claims.Count; i++)
			{
				Claim claim = placement.claims[i];
				Vector3Int a = claim.pos + domino.offset;
				for (int j = 0; j < Constants.directions.Length; j++)
				{
					Vector3Int vector3Int = a + Constants.directions[j];
					if (this.bounds.Contains(vector3Int))
					{
						Slot slot = this.GetSlot(vector3Int);
						int num = claim.keys[j];
						if (slot.keyCount[Constants.opposites[j], num].Count == 0)
						{
							return false;
						}
					}
				}
			}
			return true;
		}

		private void InitializeSeed()
		{
			System.Random random = new System.Random(this.seed + this.seed2);
			for (int i = 0; i < this.allDominos.Count; i++)
			{
				Domino domino = this.allDominos[i];
				domino.fraction = (float)random.Next() / 2.1474836E+09f;
				domino.score = domino.defaultScore * ((float)random.Next() / 2.1474836E+09f);
			}
			this.guessQueue.Clear();
			this.guessQueue.AddRange(this.allDominos);
			this.guessQueue.Sort();
		}

		private void Reset()
		{
			this.phase = MultiWave.Phase.Reset;
			this.broke++;
			for (int i = 0; i < this.slots.Length; i++)
			{
				this.slots[i].Reset();
			}
			for (int j = 0; j < this.allDominos.Count; j++)
			{
				this.allDominos[j].Reset();
			}
			for (int k = 0; k < this.cornerSlots.Length; k++)
			{
				this.cornerSlots[k].Reset();
			}
			this.removedDominos = this.removedDominosaved;
			this.placedDominos = this.placedDominosSaved;
			this.beach = 0f;
			this.coinCount = 0;
			this.onReset();
			this.openSlots.Clear();
			for (int l = 0; l < this.slots.Length; l++)
			{
				Slot slot = this.slots[l];
				if (slot.dominos.Count == 1 && slot.dominos[0].state != Domino.State.done)
				{
					this.Place(slot.dominos[0]);
				}
			}
			this.seed2++;
			this.broken = false;
			this.hasLevel = false;
		}

		private IEnumerable<bool> Resolve()
		{
			this.phase = MultiWave.Phase.Resolve;
			this.guessIndex = 0;
			this.openSlots.Clear();
			this.openBounds = default(Bounds);
			this.navigating = true;
			for (int k = 0; k < this.size.y; k++)
			{
				this.resolvedSlotsPerHeight[k] = this.savedResolvedSlotsPerHeight[k];
				this.goldPerHeight[k] = 0;
			}
			this.InitializeSeed();

			// Place first house as navigation seed
			for (int i = 0; i < this.guessQueue.Count; i++)
			{
				Domino domino = this.guessQueue[i];
				if (domino.placement.house)
				{
					if (domino.state == Domino.State.idle)
					{
						if (domino.placement.sets.Contains(this.bestHouseSet))
						{
							for (int l = 0; l < domino.placement.claims.Count; l++)
							{
								Claim claim = domino.placement.claims[l];
								if (claim.anyNavigable)
								{
									Slot slot = this.GetSlot(domino.offset + claim.pos);
									this.openSlots.Add(slot);
									slot.navigability = Slot.Navigability.Open;
									if (this.openSlots.Count == 1)
									{
										this.openBounds = new Bounds(slot.pos, Vector3.zero);
										this.navigableBounds = new Bounds(slot.pos, Vector3.zero);
									}
								}
							}
							yield return this.Place(domino);
							break;
						}
					}
				}
			}

			float verticalMult = (this.heightTier >= 5) ? this.size.y * 4f : this.size.y * 1.5f;
			float horizontalMult = (this.heightTier >= 5) ? 1.2f : 2f;
			float goldTargetPerSlot = (float)this.coinTarget / (float)this.slots.Length;
			float goldTargetPerHeight = (float)(this.coinTarget / this.size.y);

			int shortAxis = (this.size.x < this.size.z) ? 0 : 2;
			float archBiasStrength = 0.15f;
			float shortCenter = (shortAxis == 0) ? (this.size.x - 1) * 0.5f : (this.size.z - 1) * 0.5f;
			float shortMaxDist = shortCenter;

			while (this.openSlots.Count > 0)
			{
				Domino bestDomino = this.slots[0].dominos[0];
				float highestScore = float.MinValue;
				float resolvedSlots = 0f;
				for (int m = 0; m < this.resolvedSlotsPerHeight.Length; m++)
				{
					resolvedSlots += (float)this.resolvedSlotsPerHeight[m];
				}
				for (int n = 0; n < this.openSlots.Count; n++)
				{
					Slot slot2 = this.openSlots[n];
					int num = this.goldPerHeight[slot2.pos.y];
					float num2 = (float)this.resolvedSlotsPerHeight[slot2.pos.y];
					bool flag = this.coinCount < this.coinTarget && ((float)this.coinCount / resolvedSlots < goldTargetPerSlot * 1.2f || (float)num / num2 < goldTargetPerSlot * 1.5f);
					for (int num3 = 0; num3 < slot2.dominos.Count; num3++)
					{
						Domino domino4 = slot2.dominos[num3];
						if (domino4.state == Domino.State.idle)
						{
							float num4 = domino4.score;
							if (domino4.placement.house)
							{
								if (!flag)
								{
									goto IL_5E4;
								}
								num4 *= 10f;
							}
							if (this.openBounds.size != this.bounds.size && domino4.placement.navigable)
							{
								Bounds bounds = this.navigableBounds;
								Bounds bounds2 = this.openBounds;
								bounds.Encapsulate(domino4.GetNavigableBounds());
								bounds2.Encapsulate(domino4.GetOpenBounds());
								Vector3 vector = bounds.size - this.navigableBounds.size;
								if (vector.y != 0f)
								{
									num4 *= verticalMult;
								}
								if (vector.x != 0f || vector.z != 0f)
								{
									num4 *= horizontalMult;
								}
								vector = bounds2.size - this.openBounds.size;
								if (vector.y != 0f)
								{
									num4 *= verticalMult;
								}
								if (vector.x != 0f || vector.z != 0f)
								{
									num4 *= horizontalMult;
								}
							}
							if (this.topoMap != null)
							{
								Vector3Int repPos = domino4.offset;
								Bounds navBounds = domino4.placement.navigableBounds;
								int cx = repPos.x + Mathf.RoundToInt(navBounds.center.x);
								int cz = repPos.z + Mathf.RoundToInt(navBounds.center.z);
								cx = Mathf.Clamp(cx, 0, this.size.x - 1);
								cz = Mathf.Clamp(cz, 0, this.size.z - 1);
								float topoP = this.topoMap[cx, cz];
								if (domino4.placement.navigable)
									num4 *= 1f + (topoP - 0.5f) * this.topographyBiasStrength;
								else
									num4 *= 1f - (topoP - 0.5f) * this.topographyBiasStrength;
								if (num4 < 0.001f) num4 = 0.001f;
							}
							if (domino4.placement.navigable)
							{
								float shortCoord = (shortAxis == 0) ? domino4.offset.x : domino4.offset.z;
								float distFromCenter = Mathf.Abs(shortCoord - shortCenter);
								float archBonus = (distFromCenter / Mathf.Max(shortMaxDist, 1f)) * archBiasStrength;
								num4 *= 1f + archBonus;
							}
							if (num4 > highestScore)
							{
								bestDomino = domino4;
								highestScore = num4;
							}
						}
						IL_5E4:;
					}
				}
				yield return this.Place(bestDomino);
			}

			// Validate navigable bounds – must cover ground level and have meaningful area
			if (this.navigableBounds.min.y > 0f)
			{
				this.breakReason = "navigableBounds.min.y > 0";
				this.broken = true;
				yield return false;
			}
			if (this.navigableBounds.size.x <= 0f || this.navigableBounds.size.z <= 0f)
			{
				this.breakReason = "navigable bounds zero area";
				this.broken = true;
				yield return false;
			}
			if (this.beach < this.minimumBeach)
			{
				this.breakReason = "beach < minimumBeach";
				this.broken = true;
				yield return false;
			}

			this.navigating = false;
			while (this.remainingDominos > 0)
			{
				Domino domino2 = this.guessQueue[this.guessIndex];
				this.guessIndex++;
				if (domino2.state == Domino.State.idle)
				{
					if (domino2.placement.forcedNavigability)
					{
						yield return this.RemoveDomino(domino2);
					}
					else
					{
						yield return this.Place(domino2);
					}
				}
			}

			for (int j = 0; j < this.guessQueue.Count; j++)
			{
				Domino domino3 = this.guessQueue[j];
				if (domino3.state == Domino.State.done && !this.DominoFits(domino3))
				{
					this.breakReason = "domino.state == Domino.State.done && !DominoFits(domino)";
					this.broken = true;
					yield return false;
				}
			}

			yield return true;
			this.hasLevel = true;
			this.savedWave = new SavedWave(this.size, this.slots.Length);
			for (int num5 = 0; num5 < this.guessQueue.Count; num5++)
			{
				Domino domino5 = this.guessQueue[num5];
				if (domino5.state == Domino.State.done)
				{
					this.savedWave.dominos.Add(new SavedWave.SavedModule
					{
						offset = domino5.offset,
						orientedModule = domino5.GetOrientedModule(),
						placement = domino5.placement
					});
				}
			}
			yield return true;
			yield break;
		}

		public IEnumerator<GenInfo> Generate()
		{
			if (this.hasLevel)
			{
				this.Reset();
				this.hasLevel = false;
			}
			yield return new GenInfo("Resolving", GenInfo.Mode.forceInterrupt);
			bool done = false;
			while (!done)
			{
				done = true;
				using (IEnumerator<bool> enumerator = this.Resolve().GetEnumerator())
				{
					while (enumerator.MoveNext())
					{
						if (!enumerator.Current)
						{
							if (this.onBreak != null)
							{
								this.onBreak(this.breakReason);
							}
							this.Reset();
							yield return new GenInfo("Resetting", GenInfo.Mode.interruptable);
							done = false;
							break;
						}
					}
				}
			}
			yield return new GenInfo("Done", GenInfo.Mode.interruptable);
			yield break;
		}

		public void DrawGizmos()
		{
			Gizmos.matrix = this.moduleMatrix;
			if (this.openSlots != null && this.openSlots.Count > 0)
			{
				Gizmos.color = Color.green.SetA(0.2f);
				Gizmos.DrawWireCube(this.openBounds.center, this.openBounds.size + Vector3.one);
			}
			foreach (Slot slot in this.slots)
			{
				if (slot.navigability != Slot.Navigability.Unclear)
				{
					Gizmos.color = ((slot.navigability != Slot.Navigability.Open) ? Color.green : Color.red) * 2f;
					Gizmos.DrawSphere(slot.pos, 0.1f);
				}
			}
		}

		/// <summary>
		/// Deterministically generates a valid island size (square footprint) that respects
		/// the hard limit of 512 tiles, max width 16 and max height 8.
		/// </summary>
		private static Vector3Int GenerateSizeFromSeeds(int seed, int seed2)
		{
			System.Random rng = new System.Random(seed ^ seed2);
			// Collect all valid (width, height) pairs
			List<Vector2Int> candidates = new List<Vector2Int>();
			for (int w = 6; w <= 16; w++)
			{
				for (int h = 1; h <= 8; h++)
				{
					if (w * w * h <= 512)
					{
						candidates.Add(new Vector2Int(w, h));
					}
				}
			}
			// Weight by volume – larger islands are more fun, but still allow variety
			float totalWeight = 0f;
			foreach (var c in candidates)
			{
				totalWeight += c.x * c.x * c.y; // volume weight
			}
			float roll = (float)rng.NextDouble() * totalWeight;
			Vector2Int chosen = candidates[0];
			foreach (var c in candidates)
			{
				float w = c.x * c.x * c.y;
				if (roll < w)
				{
					chosen = c;
					break;
				}
				roll -= w;
			}
			return new Vector3Int(chosen.x, chosen.y, chosen.x);
		}

		public string name;
		public Vector3Int size;
		public float minimumBeach;
		public float beach;
		public int seed;
		public int seed2;
		public Bounds bounds = default(Bounds);
		public bool hasLevel;
		public List<Domino> allDominos;
		public List<Domino> guessQueue;
		private bool broken;
		private string breakReason;
		public Action<string> onBreak = delegate(string A_0) { };
		private StraightMarcher marcher;
		public float visibilityThreshold = 0.4f;
		[Header("Counts")]
		public int broke;
		public Action onReset = delegate() { };
		public Action<MultiWave> onSave = delegate(MultiWave A_0) { };
		public int removedDominos;
		public int removedDominosaved;
		public int placedDominos;
		public int placedDominosSaved;
		[Header("Lists")]
		public int queueCount;
		public int startSum;
		public int currentSum;
		public Slot[] slots;
		public CornerSlot[] cornerSlots;
		public bool navigating;
		public List<Slot> openSlots;
		private int[] goldPerHeight;
		private int[] resolvedSlotsPerHeight;
		private int[] savedResolvedSlotsPerHeight;
		private Bounds openBounds;
		private Bounds navigableBounds;
		public SavedWave savedWave;
		private List<string> tilesetKeys;
		private int coinTarget;
		private int coinCount;
		public int guessIndex;
		public int guessQueueLength;
		private ModuleSet bestHouseSet;
		public int heightTier;
		private float[,] topoMap;
		private float topographyBiasStrength;

		public enum Phase
		{
			Setup,
			InitialContradictions,
			PlacingObvious,
			SavingState,
			Reset,
			Resolve
		}
	}
}
```
islandsizefield
```csharp
using System;
using UnityEngine;

namespace Voxels.TowerDefense.CampaignGeneration
{
	[Serializable]
	public class IslandSizeField
	{
		public IslandSizeField(int minWidth = 6, int maxWidth = 11, int minHeight = 1, int maxHeight = 4)
		{
			int num = maxWidth - minWidth + 1;
			int num2 = maxHeight - minHeight + 1;
			int num3 = num * num2;
			this.ranges = new Vector2[num3];
		}

		/// <summary>
		/// Height tier is now the actual chosen height (1‑8).
		/// </summary>
		public int HeightTier { get; private set; }

		public float GetAverageWidth()
		{
			float num = 0f;
			float num2 = 0f;
			for (int i = 0; i < this.ranges.Length; i++)
			{
				int num3 = i % 6 + 6;
				float num4 = this.ranges[i].y - this.ranges[i].x;
				num += (float)num3 * num4;
				num2 += num4;
			}
			return num / num2;
		}

		/// <summary>
		/// Returns random square dimensions (width = depth) that never exceed the
		/// hard 512‑tile volume limit. Width is capped at 16, height at 8.
		/// The input fraction is used together with a deterministic seed to ensure
		/// the same result on every reload.
		/// </summary>
		public Vector2Int GetRandomDimensions(float fraction)
		{
			// Deterministic random from fraction
			uint state = (uint)(fraction * 4294967295.0);
			state = state * 1103515245 + 12345;

			// Build list of all valid (width, height) pairs
			var valid = new System.Collections.Generic.List<Vector2Int>();
			for (int w = 6; w <= 16; w++)
			{
				for (int h = 1; h <= 8; h++)
				{
					if (w * w * h <= 512)
					{
						valid.Add(new Vector2Int(w, h));
					}
				}
			}

			// Use PRNG to pick one, weighting by area (larger islands more likely)
			float totalWeight = 0f;
			foreach (var v in valid)
				totalWeight += v.x * v.x * v.y;
			state = state * 1103515245 + 12345;
			float roll = ((float)(state % 10000) / 10000f) * totalWeight;
			Vector2Int chosen = valid[0];
			foreach (var v in valid)
			{
				float w = v.x * v.x * v.y;
				if (roll < w)
				{
					chosen = v;
					break;
				}
				roll -= w;
			}

			this.HeightTier = chosen.y;
			return chosen;
		}

		public const int minWidth = 6;
		public const int minHeight = 1;
		public const int maxHeight = 4;
		public const int maxWidth = 11;
		public const int sizeX = 6;
		[SerializeField] public Vector2[] ranges;
	}
}
```
agent
```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using ReflexCLI.Attributes;
using UnityEngine;
using UnityEngine.EventSystems;
using Voxels.TowerDefense.SpriteMagic;
using Voxels.TowerDefense.TriFlow;

namespace Voxels.TowerDefense
{
    [SelectionBase]
    public class Agent : MonoBehaviour, IPassedClick
    {
        public AgentState rootState
        {
            get { return this.stateRoot.rootState; }
        }

        public bool isEnglish
        {
            get { return this.faction.side == Faction.Side.English; }
        }

        public bool isViking
        {
            get { return this.faction.side == Faction.Side.Viking; }
        }

        public SphereCollider col
        {
            get
            {
                if (this._col == null)
                {
                    this._col = base.GetComponent<SphereCollider>();
                }
                return this._col;
            }
        }

        public Body body { get; private set; }

        public Animator animator { get; private set; }

        public SpriteAnimator spriteAnimator { get; private set; }

        public Ragdoller ragdoller { get; private set; }

        public Stun stun { get; private set; }

        public Vector3 lPos
        {
            get { return this.navPos.pos; }
        }

        public Vector3 wChestPos
        {
            get { return this.wPos + this.chestOffset; }
        }

        public Vector3 chestPos
        {
            get { return base.transform.position + this.chestOffset; }
        }

        public Vector3 chestOffset
        {
            get { return Vector3.up * this.chestHeight; }
        }

        public float chestHeight
        {
            get { return 0.23f * base.transform.localScale.y; }
        }

        public float totalIntimidation
        {
            get { return this.hardIntimidation + this.softIntimidation; }
        }

        public float maxIntimidation
        {
            get { return Mathf.Max(this.hardIntimidation, this.softIntimidation); }
        }

        public bool intimidated
        {
            get { return this.totalIntimidation > 0f; }
        }

        public Vector3 lookDir
        {
            get { return (!this.navPos.valid) ? base.transform.forward : Vector3.Cross(base.transform.right, this.navPos.GetNormal()); }
        }

        public float animationTime
        {
            get { return this.animator.GetCurrentAnimatorStateInfo(0).normalizedTime; }
        }

        public bool animationDone
        {
            get { return !this.animationChangeFrame && this.animationTime >= 1f; }
        }

        public void Throw()
        {
            if (this.onThrow != null)
            {
                this.onThrow();
                this.onThrow = null;
            }
        }

        public void StartThrow()
        {
            if (this.onStartThrow != null)
            {
                this.onStartThrow();
                this.onStartThrow = null;
            }
        }

        public void PlayAnimation(string anim)
        {
            this.PlayAnimation(Animator.StringToHash(anim));
        }

        public void PlayAnimation(int anim)
        {
            this.animator.Play(anim);
            this.animationChangeFrame = true;
        }

        public void PlayAnimation(int anim, float normalizedTime)
        {
            this.animator.Play(anim, -1, normalizedTime);
            this.animationChangeFrame = true;
        }

        public bool moveAnimate
        {
            get { return this.moveAnimateState.active; }
            set { this.moveAnimateState.SetActive(value); }
        }

        public float healthFraction
        {
            get { return this.health / this.maxHealth; }
        }

        public void FillHealth()
        {
            this.health = this.maxHealth;
        }

        public float area
        {
            get { return this.radius * this.radius * 3.2f; }
        }

        public float radius
        {
            get { return this.scale * 0.12f; }
        }

        public float mass
        {
            get { return this.scale * this.scale * this.scale; }
        }

        public float scale
        {
            get { return base.transform.localScale.x; }
            set { base.transform.localScale = value * Vector3.one; }
        }

        public event Agent.MeleeBrainDelegate OnAttackTelegraphed = delegate(Brain A_0) { };

        public event Agent.SelectedDelegate OnAgentSelected = delegate(Agent A_0, bool A_1, bool A_2) { };

        public float friendRatio
        {
            get
            {
                if (this._eRatio == null)
                {
                    this._eRatio = new FrameCache<float>(() => this.GetFriendRatio());
                }
                return this._eRatio.value;
            }
        }

        public float enemyRatio
        {
            get { return 1f - this.friendRatio; }
        }

        public ITriFlowObject enemyEntity
        {
            get { return this.enemyData.entity; }
        }

        public Agent enemyAgent
        {
            get
            {
                Agent agent = this.enemyData.agent;
                return (!agent || !agent.navPos.valid) ? null : agent;
            }
        }

        public Brain enemyBrain
        {
            get
            {
                Agent agent = this.enemyData.agent;
                return (!agent || !agent.navPos.valid) ? null : agent.brain;
            }
        }

        public void Setup(NavPos navPos)
        {
            base.transform.SetParent(navPos.transform);
            this.wPos = navPos.wPos;
            base.transform.localPosition = navPos.pos;
            this.beats = new LazyBeats(100f);
            this.health = this.maxHealth;
            this.body = base.GetComponent<Body>();
            this.ragdoller = base.GetComponent<Ragdoller>();
            this.stun = base.GetComponent<Stun>();
            this.animator = base.GetComponent<Animator>();
            this.spriteAnimator = base.GetComponentInChildren<SpriteAnimator>();
            this.batchedShadow = this.shadow.GetComponent<BatchedSprite>();
            this.agentComponents = base.GetComponentsInChildren<AgentComponent>(true);
            this.attackResponders = base.GetComponentsInChildren<IAttackResponder>(true).ToList<IAttackResponder>();
            this.postAttacks = base.GetComponentsInChildren<Agent.IPostAttack>(true);
            this.brain = base.GetComponentInChildren<Brain>(true);
            this.uniqueDebugColor = ((!this.isEnglish) ? this.faction.color : ((this.squad as EnglishSquad).hero.color * 1.5f));
            this.navPos = navPos;
            this.rangeWorry = new Worry(this);
            this.personality.speed = UnityEngine.Random.Range(0.9f, 1.1f);
            this.personality.reflex = UnityEngine.Random.Range(0.85f, 1.25f);
            this.personality.boldness = UnityEngine.Random.value;
            this.personality.wanderlust = UnityEngine.Random.value;
            this.personality.restlessness = UnityEngine.Random.Range(0.4f, 1f);
            this.personality.phase = UnityEngine.Random.Range(0f, 6.2831855f);
            this.strafeSeed = UnityEngine.Random.Range(0f, 6.2831855f);
            this.scoutState = Agent.ScoutState.Following;
            this.nextScoutRoll = Time.time + UnityEngine.Random.Range(2f, 6f);
            this.previousOrderDist = this.orderDist;
            this.aliveAndGrounded = new AgentState("AliveAndGrounded", this.rootState, false, false);
            this.spawned = new AgentState("Spawned", this.rootState, false, false);
            this.exclusives = new AgentState("Exclusives", this.aliveAndGrounded, false, false);
            this.lifeState = new AgentState("Life", this.rootState, false, false);
            this.deadState = new AgentState("Dead", this.lifeState, false, true);
            this.aliveState = new AgentState("Alive", this.lifeState, true, true);
            this.navigationState = new AgentState("Navigation", this.rootState, true, false);
            this.groundedState = new AgentState("Grounded", this.navigationState, true, true);
            this.moveAnimateState = new AgentState("MoveAnimate", this.rootState, true, true);
            AgentState agentState = this.aliveAndGrounded;
            agentState.OnEmpty = (Action)Delegate.Combine(agentState.OnEmpty, new Action(this.exclusives.SetActiveTrue));
            this.aliveAndGrounded.OnUpdate += this.UpdateColor;
            AgentState agentState2 = this.groundedState;
            agentState2.OnDeactivate = (Action)Delegate.Combine(agentState2.OnDeactivate, new Action(this.UpdateColor));
            AgentState agentState3 = this.groundedState;
            agentState3.OnDeactivate = (Action)Delegate.Combine(agentState3.OnDeactivate, new Action(delegate()
            {
                this.navPos.SetNull();
            }));
            AgentState agentState4 = this.aliveState;
            agentState4.OnChange = (Action<bool>)Delegate.Combine(agentState4.OnChange, new Action<bool>(this.AliveAndGrounded));
            AgentState agentState5 = this.groundedState;
            agentState5.OnChange = (Action<bool>)Delegate.Combine(agentState5.OnChange, new Action<bool>(this.AliveAndGrounded));
            AgentState agentState6 = this.spawned;
            agentState6.OnChange = (Action<bool>)Delegate.Combine(agentState6.OnChange, new Action<bool>(this.AliveAndGrounded));
            AgentState agentState7 = this.groundedState;
            agentState7.OnChange = (Action<bool>)Delegate.Combine(agentState7.OnChange, new Action<bool>(this.shadow.gameObject.SetActive));
            AgentState agentState8 = this.aliveAndGrounded;
            agentState8.OnDeactivate = (Action)Delegate.Combine(agentState8.OnDeactivate, new Action(delegate()
            {
                this.enemyDist = 0f;
                this.enemyDir = Vector3.zero;
                this.enemyData = Data.empty;
                this.orderDir = Vector3.zero;
                this.orderDist = 0f;
            }));
            IAgentSetup[] componentsInChildren = base.GetComponentsInChildren<IAgentSetup>();
            for (int i = 0; i < componentsInChildren.Length; i++)
            {
                componentsInChildren[i].Setup(this);
            }
            AgentState agentState9 = this.groundedState;
            agentState9.OnActivate = (Action)Delegate.Combine(agentState9.OnActivate, new Action(delegate()
            {
                if (!this.navPos.valid)
                {
                    UnityEngine.Debug.LogError("Trying to set grounded state active without a valid nav pos!", this);
                }
            }));
            AgentState agentState10 = this.spawned;
            agentState10.OnChange = (Action<bool>)Delegate.Combine(agentState10.OnChange, new Action<bool>(delegate(bool x)
            {
                if (x)
                {
                    this.rootState.UpdateEmpties();
                    Agent.allActive.Add(this);
                    this.faction.agents.Add(this);
                }
                else
                {
                    Agent.allActive.Remove(this);
                    this.faction.agents.Remove(this);
                }
            }));
            this.deadState.OnUpdate += delegate()
            {
                if (this.aliveAndGrounded.active)
                {
                    this.FinalDeath();
                    return;
                }
                if (this.deadState.timeSinceActivation > 8f)
                {
                    this.FinalDeath();
                    return;
                }
            };
            AgentState agentState11 = this.aliveAndGrounded;
            agentState11.OnActivate = (Action)Delegate.Combine(agentState11.OnActivate, new Action(delegate()
            {
                if (this.deadState.active)
                {
                    this.FinalDeath();
                    return;
                }
            }));
        }

        public void Spawn()
        {
            this.spawned.SetActive(true);
        }

        public float GetFriendRatio()
        {
            float num = this.faction.presence.SampleAmount(this.navPos);
            float num2 = this.faction.enemy.presence.SampleAmount(this.navPos);
            return num / (num + num2);
        }

        [ContextMenu("IsDangerous?")]
        private void PrintDangerous()
        {
            UnityEngine.Debug.Log("Dangerous = " + this.dangerous);
        }

        public void SetDangerous(bool value)
        {
            this.dangerous = value;
        }

        public void SetDangerousInverse(bool value)
        {
            this.dangerous = !value;
        }

        public void SetMoveAnimate(bool value)
        {
            this.moveAnimate = value;
        }

        public void SetMoveAnimateInverse(bool value)
        {
            this.moveAnimate = !value;
        }

        public void SetCollider(bool value)
        {
            this.col.enabled = value;
        }

        public void SetColliderInverse(bool value)
        {
            this.SetCollider(!value);
        }

        public void TelegraphAttack(Brain attacker)
        {
            this.OnAttackTelegraphed(attacker);
        }

        public bool DealDamage(Attack attack)
        {
            bool result;
            for (int i = 0; i < this.attackResponders.Count; i++)
            {
                this.attackResponders[i].ModifyAttack(ref attack);
            }
            if (attack.ignore)
            {
                result = false;
            }
            else
            {
                float num = attack.knockback / this.mass;
                Vector3 normalized = attack.direction.normalized;
                Vector3 vector = normalized * num;
                float num2 = Vector3.Dot(this.velocity + vector, normalized);
                if (num2 > num)
                {
                    vector -= normalized * (num2 - num);
                }
                if (attack.launchImpulse > 0f && this.ragdoller)
                {
                    float num3 = attack.launchImpulse / (this.scale * this.scale);
                    if (num3 >= 0.9f)
                    {
                        Vector3 vector2 = this.velocity + normalized * num;
                        vector2.y = Mathf.Max(vector2.y, 0f) + num3;
                        this.ragdoller.Launch(vector2);
                    }
                }
                bool flag = false;
                if (this.aliveState.active)
                {
                    if (attack.damage > 0f || attack.knockback > 0f)
                    {
                        this.health -= attack.damage;
                    }
                    if (this.health <= 0f)
                    {
                        flag = true;
                        attack.soundSuffix = "Kill";
                        attack.effect = ScriptableObjectSingleton<PrefabManager>.instance.bloodSlash;
                        this.col.radius *= 0.5f;
                        this.deadState.SetActive(true);
                        if (this.squad)
                        {
                            this.squad.ReportDead(this);
                        }
                    }
                    if (!flag)
                    {
                        bool flag2 = true;
                        int num4 = 0;
                        while (num4 < this.postAttacks.Length && flag2)
                        {
                            this.postAttacks[num4].PostAttack(attack, num, ref flag2);
                            num4++;
                        }
                        IslandGameplayManager.RequestCombatAudio(this.hurtSound, base.gameObject);
                    }
                }
                else
                {
                    vector /= 2f;
                }
                this.velocity += vector;
                if (attack.effect != null)
                {
                    attack.effect.PlayAt(attack.pos, -attack.direction.GetZeroY());
                }
                if (attack.hasSound)
                {
                    IslandGameplayManager.RequestCombatAudio(attack.soundPrefix, attack.soundSuffix, base.gameObject);
                }
                result = flag;
            }
            return result;
        }

        public void DoAnimationAction()
        {
            if (this.animationAction != null)
            {
                this.animationAction();
            }
            this.animationAction = null;
        }

        public void PlayAnimation(int anim, Action animationAction)
        {
            this.animationAction = animationAction;
            this.animator.Play(anim);
            this.animationChangeFrame = true;
        }

        public void KillImmediate()
        {
            this.squad.ReportDead(this);
            this.FinalDeath();
        }

        public void FinalDeath()
        {
            this.onFinalDeath();
            UnityEngine.Object.Destroy(base.gameObject);
        }

        public void SetSelection(bool selected, bool showGodray)
        {
            this.OnAgentSelected(this, selected, showGodray);
        }

        private void AliveAndGrounded(bool b)
        {
            this.aliveAndGrounded.SetActive(this.aliveState.active && this.groundedState.active && this.spawned.active);
        }

        public bool IsPathing()
        {
            return this.navPos.valid && this.brain.brainState.active;
        }

        public void SetLevel(int level)
        {
            if (this.squad is EnglishSquad)
            {
                EnglishSquad englishSquad = this.squad as EnglishSquad;
                this.scale = 1f + ((float)(englishSquad.level + englishSquad.squadTemplate.level) - 2f) * 0.06f;
            }
            foreach (ILevelComponent levelComponent in base.GetComponentsInChildren<ILevelComponent>(true))
            {
                levelComponent.OnSetLevel(this, level);
            }
        }

        private void OnDestroy()
        {
            if (this.rootState != null)
            {
                this.rootState.SetActive(false);
                this.rootState.OnDestroy();
            }
            if (this.squad)
            {
                this.squad.ReportDestroyed(this);
            }
            if (this.faction)
            {
                this.faction.agents.Remove(this);
            }
            Agent.allActive.Remove(this);
            this.faction = null;
            this.squad = null;
            this.body = null;
            this.ragdoller = null;
            this.stun = null;
            this._col = null;
            this.animator = null;
            this.spriteAnimator = null;
            this.agentComponents = null;
            this.attackResponders = null;
            this.postAttacks = null;
            this.aliveAndGrounded = null;
            this.spawned = null;
            this.exclusives = null;
            this.navigationState = null;
            this.groundedState = null;
            this.moveAnimateState = null;
            this.lifeState = null;
            this.deadState = null;
            this.aliveState = null;
            this.onFinalDeath = null;
            this.OnAttackTelegraphed = null;
            this.OnAgentSelected = null;
            this.brain = null;
            this._eRatio = null;
            this.enemyData = default(Data);
            this.navPos = default(NavPos);
            this.onThrow = null;
            this.onStartThrow = null;
            this.look = default(Agent.Look);
        }

        public void SetDirection(Vector3 dir)
        {
            dir.y = 0f;
            if (float.IsNaN(dir.x) || (double)dir.sqrMagnitude < 1E-05)
            {
                return;
            }
            base.transform.rotation = Quaternion.LookRotation(dir);
        }

        public void SetNavPos(NavPos newNavPos)
        {
            this.navPos = newNavPos;
        }

        public void LookAt(Vector3 pos, float speed = 720f, float threshold = 20f)
        {
            Vector3 dir = pos - this.wPos;
            this.look.Set(dir, speed * this.personality.reflex, threshold);
        }

        public bool TriCast(Agent otherAgent)
        {
            return otherAgent && otherAgent.aliveAndGrounded.active && this.navPos.TriCast(otherAgent.navPos);
        }

        public void LookInDirection(Vector3 dir, float speed = 720f, float threshold = 20f)
        {
            this.look.Set(dir, speed * this.personality.reflex, threshold);
        }

        private void ApplyLook()
        {
            Vector3 zeroY = this.look.dir.GetZeroY();
            if (zeroY == Vector3.zero || float.IsNaN(zeroY.x))
            {
                return;
            }
            Quaternion quaternion = Quaternion.LookRotation(zeroY);
            if (Quaternion.Angle(quaternion, base.transform.rotation) < this.look.threshold)
            {
                return;
            }
            base.transform.rotation = Quaternion.RotateTowards(base.transform.rotation, quaternion, this.look.speed * Time.deltaTime);
        }

        private void MaybeFixNavPos()
        {
            if (!this.navPos.valid)
            {
                UnityEngine.Debug.LogError("Agent navPos invalid for unknown reason, finding new navPos");
                this.navPos = new NavPos(this.faction.island.navMesh, base.transform.position, true, 1f);
            }
        }

        public void Intimidate(float soft, float hard)
        {
            float resolve = Mathf.Lerp(1.3f, 0.7f, this.personality.boldness);
            this.softIntimidation = Mathf.Max(this.softIntimidation, soft * resolve);
            this.hardIntimidation = Mathf.Max(this.hardIntimidation, hard * resolve);
        }

        private void UpdateColor()
        {
            Color color = this.spriteAnimator.color;
            color.b = 1f - this.healthFraction;
            this.spriteAnimator.color = color;
        }

        private void FixedUpdateAgent(float dt, float forceDt, float velocityScale)
        {
            this.animationChangeFrame = false;
            this.walkDir = Vector3.zero;
            this.movability = 1f;
            this.enemyMovability = 1f;
            this.faction.enemy.presence.SampleFullData(this.navPos, ref this.enemyDist, ref this.enemyDir, ref this.enemyData);
            if (this.navPos.valid)
            {
                this.brain.order.SampleOrder(this.navPos, ref this.orderDir, ref this.orderDist);
            }
            else
            {
                this.orderDist = 100f;
                this.orderDir = Vector3.zero;
            }
            this.stateRoot.Update();
            this.UpdateAutonomousMovement();
            if (this.groundedState.active)
            {
                this.MaybeFixNavPos();
                Vector3 vector = this.navPos.pos;
                float sqrMagnitude = this.walkDir.sqrMagnitude;
                if (float.IsNaN(sqrMagnitude) || float.IsInfinity(sqrMagnitude))
                {
                    this.walkDir = Vector3.zero;
                }
                else if (sqrMagnitude != 0f && this.body.hopping.active)
                {
                    Vector3 normalized = this.walkDir.GetZeroY().normalized;
                    Vector3 slope = this.navPos.GetSlope();
                    float num = Vector3.Dot(normalized, slope);
                    if (num > 0f)
                    {
                        this.speed *= ExtraMath.RemapValue(num, 0f, 0.2f, 1f, 1.4f);
                    }
                    else if (num < 0f)
                    {
                        this.speed *= ExtraMath.RemapValue(num, -0.2f, 0f, 0.6f, 1f);
                    }
                    float value = Vector3.Dot(normalized, base.transform.forward);
                    this.speed *= ExtraMath.RemapValue(value, -1f, 1f, 0.7f, 1.1f);
                    Vector3 clampedMagnitude = this.walkDir.GetClampedMagnitude(dt * this.speed);
                    vector += clampedMagnitude;
                }
                if (!float.IsNaN(this.force.sqrMagnitude))
                {
                    vector += this.force * forceDt;
                }
                this.navPos.pos = vector;
                if (this.velocity != Vector3.zero)
                {
                    if (float.IsNaN(this.velocity.sqrMagnitude))
                    {
                        this.velocity = Vector3.zero;
                    }
                    Vector3 up = this.navPos.tri.up;
                    if (!this.navPos.onMain)
                    {
                        this.navPos.transform.TransformVector(up);
                    }
                    float num2 = Vector3.Dot(up, this.velocity);
                    if (num2 < 0f)
                    {
                        this.velocity -= up * num2;
                    }
                    Vector3 vector2 = (!this.navPos.onMain) ? this.navPos.transform.InverseTransformVector(this.velocity) : this.velocity;
                    Vector3 target = this.navPos.pos + vector2 * dt;
                    Edge edge;
                    this.navPos.MoveTo(target, out edge);
                    if (edge != null)
                    {
                        if (edge.cliff || !this.navPos.navigationMesh.island)
                        {
                            float num3 = Vector3.Dot(vector2, -edge.borderVector);
                            if (!this.ragdoller || !this.ragdoller.MaybeRagdoll(num3))
                            {
                                vector2 -= edge.borderVector * Vector3.Dot(vector2, edge.borderVector);
                                this.velocity = this.navPos.transform.TransformVector(vector2);
                            }
                        }
                        else
                        {
                            vector2 = Vector3.Reflect(vector2, edge.borderVector) * 0.5f;
                            this.velocity = this.navPos.transform.TransformVector(vector2);
                        }
                    }
                    this.velocity *= velocityScale;
                }
                if (this.groundedState.active)
                {
                    this.wPos = this.navPos.wPos;
                }
                this.force = Vector3.zero;
                this.speed = this.maxSpeed * this.personality.speed;
                this.ApplyLook();
            }
            else
            {
                this.wPos = base.transform.position;
            }
            this.look = default(Agent.Look);
            this.health = Mathf.MoveTowards(this.health, this.maxHealth, dt / 60f);
            this.rangeWorry.Update();
        }

        private void UpdateAutonomousMovement()
        {
            if (!this.aliveAndGrounded.active)
            {
                this.scoutState = Agent.ScoutState.Following;
                this.previousOrderDist = this.orderDist;
                return;
            }

            bool orderChanged = Mathf.Abs(this.orderDist - this.previousOrderDist) > 0.3f;
            this.previousOrderDist = this.orderDist;

            bool stopScouting = this.enemyDist < 4f || this.intimidated || !this.moveAnimate || orderChanged || this.IsSquadInCombat();

            if (this.scoutState != Agent.ScoutState.Following && stopScouting)
            {
                this.scoutState = Agent.ScoutState.Following;
                this.nextScoutRoll = Time.time + UnityEngine.Random.Range(1.5f, 3f);
                return;
            }

            bool safeAndClear = this.enemyDist > 5f && !this.intimidated && this.moveAnimate
                                && (this.brain == null || !this.brain.actingState.active)
                                && !this.IsSquadInCombat()
                                && !orderChanged;

            if (this.scoutState == Agent.ScoutState.Following && safeAndClear)
            {
                this.TryStartScouting();
            }

            if (this.scoutState == Agent.ScoutState.Moving)
            {
                this.UpdateScoutMoving();
            }
            else if (this.scoutState == Agent.ScoutState.Paused)
            {
                this.UpdateScoutPaused();
            }
            else if (this.moveAnimate && this.enemyDist < 2.5f)
            {
                this.ApplyCombatFlavor();
            }
        }

        private void TryStartScouting()
        {
            if (Time.time < this.nextScoutRoll)
                return;

            this.nextScoutRoll = Time.time + UnityEngine.Random.Range(3f, 7f);

            if (this.personality.wanderlust > 0.7f && UnityEngine.Random.value < 0.4f)
            {
                this.scoutSessionEnd = Time.time + UnityEngine.Random.Range(5f, 11f);
                this.PickScoutTarget();
            }
        }

        private void PickScoutTarget()
        {
            Vector2 rand2 = UnityEngine.Random.insideUnitCircle;
            Vector3 dir = new Vector3(rand2.x, 0f, rand2.y);
            if (dir.sqrMagnitude < 0.0001f)
                dir = base.transform.forward;
            dir.Normalize();

            float dist = Mathf.Lerp(1.5f, 5f, this.personality.wanderlust);
            this.scoutTarget = this.navPos.pos + dir * dist;
            this.scoutState = Agent.ScoutState.Moving;
            this.scoutTimer = Time.time + Mathf.Lerp(6f, 2.5f, this.personality.restlessness);
        }

        private void UpdateScoutMoving()
        {
            Vector3 toTarget = this.scoutTarget - this.navPos.pos;
            toTarget.y = 0f;
            if (toTarget.sqrMagnitude > 0.0025f)
            {
                this.walkDir = toTarget.normalized * 0.55f;
                this.LookInDirection(toTarget, 200f, 30f);
            }
            if (toTarget.sqrMagnitude < 0.04f || Time.time > this.scoutTimer)
            {
                this.scoutState = Agent.ScoutState.Paused;
                this.scoutTimer = Time.time + Mathf.Lerp(2.5f, 0.7f, this.personality.restlessness);
            }
        }

        private void UpdateScoutPaused()
        {
            this.walkDir = Vector3.zero;
            float t = Time.time * 0.6f + this.personality.phase;
            this.LookInDirection(new Vector3(Mathf.Sin(t), 0f, Mathf.Cos(t)), 60f, 5f);
            if (Time.time > this.scoutTimer)
            {
                if (Time.time > this.scoutSessionEnd || UnityEngine.Random.value > this.personality.wanderlust)
                {
                    this.scoutState = Agent.ScoutState.Following;
                }
                else
                {
                    this.PickScoutTarget();
                }
            }
        }

        private void ApplyCombatFlavor()
        {
            float t = Time.time * (1.3f + this.personality.boldness) + this.strafeSeed;
            Vector3 lateral = Vector3.Cross(Vector3.up, this.enemyDir.GetZeroY());
            float weave = Mathf.Sin(t) * Mathf.Lerp(0.15f, 0.4f, this.personality.boldness);
            this.walkDir += lateral * weave;
            this.walkDir += this.enemyDir.GetZeroY() * Mathf.Lerp(-0.12f, 0.08f, this.personality.boldness);
        }

        private bool IsSquadInCombat()
        {
            if (this.squad == null || this.squad.agents == null)
                return false;
            foreach (Agent a in this.squad.agents)
            {
                if (a == this || !a.aliveAndGrounded.active)
                    continue;
                if (a.enemyDist < 4f || a.intimidated)
                    return true;
            }
            return false;
        }

        public static void UpdateAllAgents(float dt, float forceDt)
        {
            using (new ScopedProfiler("UpdateAllAgents", null))
            {
                for (int i = 0; i < Agent.allActive.Count; i++)
                {
                    Agent.allActive[i].hardIntimidation = 0f;
                    Agent.allActive[i].softIntimidation = 0f;
                }
                for (int j = 0; j < Agent.allActive.Count; j++)
                {
                    Agent agent = Agent.allActive[j];
                    if (agent.aliveAndGrounded.active && agent.enemyEntity != null && !agent.enemyEntity.Equals(null))
                    {
                        agent.enemyEntity.OnProximity(agent);
                        Agent enemyAgent = agent.enemyAgent;
                        if (enemyAgent && enemyAgent.aliveAndGrounded.active && enemyAgent.enemyAgent != agent)
                        {
                            agent.brain.OnProximity(enemyAgent);
                        }
                    }
                }
                float velocityScale = 0.9f;
                using (new ScopedProfiler("FixedUpdateAgent", null))
                {
                    for (int k = 0; k < Agent.allActive.Count; k++)
                    {
                        try
                        {
                            Agent agent2 = Agent.allActive[k];
                            if (agent2)
                            {
                                agent2.FixedUpdateAgent(dt, forceDt, velocityScale);
                            }
                        }
                        catch (Exception exception)
                        {
                            Agent agent3 = Agent.allActive[k];
                            UnityEngine.Debug.LogException(exception);
                            EnglishSquad englishSquad = agent3.squad as EnglishSquad;
                            if (!englishSquad || englishSquad.heroAgent != agent3)
                            {
                                agent3.FinalDeath();
                            }
                        }
                    }
                }
            }
        }

        public void OnPassedClick(ClickPasser clickPasser, PointerEventData eventData, RaycastHit raycastHit)
        {
            if (this.squad && this.squad.passedClicker != null)
            {
                this.squad.passedClicker.OnPassedClick(clickPasser, eventData, raycastHit);
            }
        }

        private void OnDrawGizmos()
        {
            if (!Application.isPlaying)
            {
                this.wPos = base.transform.position;
            }
            Gizmos.matrix = ExtraGizmos.CloserToCameraMatrix();
            Gizmos.color = this.uniqueDebugColor;
            ExtraGizmos.DrawCircle(base.transform.position, this.radius, 8);
            Gizmos.DrawRay(base.transform.position, this.lookDir * this.radius);
            Gizmos.DrawLine(base.transform.position, this.chestPos);
            Gizmos.color *= 2f;
            Gizmos.DrawSphere(base.transform.position, this.radius / 4f);
        }

        private void OnDrawGizmosSelected()
        {
            if (Application.isPlaying && this.navPos.valid)
            {
                Gizmos.color = this.faction.color;
                Gizmos.DrawLine(base.transform.position, this.wPos);
                Gizmos.color *= 2f;
                Gizmos.DrawSphere(this.wPos, this.radius / 8f);
                Gizmos.color = new Color(1f, 0.8f, 0.6f);
                if (this.enemyAgent)
                {
                    Gizmos.DrawLine(base.transform.position, this.enemyAgent.transform.position);
                }
                Gizmos.color = Color.yellow;
                if (this.rangeWorry.valid)
                {
                    Gizmos.DrawRay(base.transform.position, this.rangeWorry.dir.normalized);
                }
            }
        }

        public T GetOrAddComponent<T>() where T : AgentComponent
        {
            T t = base.GetComponent<T>();
            if (!t)
            {
                t = base.gameObject.AddComponent<T>();
                t.Setup(this);
                if (this.spawned != null)
                {
                }
            }
            return t;
        }

        public void UpdateVisuals()
        {
            foreach (Agent.IUpdateVisuals updateVisuals in base.GetComponentsInChildren<Agent.IUpdateVisuals>(true))
            {
                updateVisuals.UpdateVisuals();
            }
        }

        [Conditional("UNITY_EDITOR")]
        [Conditional("DEVELOPMENT_BUILD")]
        private void LogStateError(int chn, string msg)
        {
            UnityEngine.Debug.LogErrorFormat("{0} - {1}", new object[] { msg, base.name });
            StackTrace stackTrace = new StackTrace();
        }

        private static List<Agent> allActive = new List<Agent>();

        [SerializeField]
        private AgentStateRoot stateRoot = new AgentStateRoot(16);

        public Faction faction;

        public NavPos navPos;

        public float outwardness = 0.5f;

        public Squad squad;

        public float walkedDistance;

        private SphereCollider _col;

        public AgentComponent[] agentComponents;

        public List<IAttackResponder> attackResponders;

        public Agent.IPostAttack[] postAttacks;

        public Vector3 wPos;

        public float movability = 1f;

        public float enemyMovability = 1f;

        public Vector3 velocity;

        public Vector3 walkDir;

        public Vector3 force;

        public float hardIntimidation;

        public float softIntimidation;

        private bool animationChangeFrame;

        public FabricEventReference hurtSound = string.Empty;

        private Agent.Look look = default(Agent.Look);

        public SpriteRenderer shadow;

        [NonSerialized]
        public BatchedSprite batchedShadow;

        public Action onStartThrow;

        public Action onThrow;

        public AgentState aliveAndGrounded;

        public AgentState spawned;

        public AgentState exclusives;

        public AgentState navigationState;

        public AgentState groundedState;

        public AgentState moveAnimateState;

        public AgentState lifeState;

        public AgentState deadState;

        public AgentState aliveState;

        [Header("Worries")]
        public Worry rangeWorry;

        [Header("Flags")]
        public bool shield;

        public float maxHealth = 1f;

        public float health = 1f;

        [ConsoleCommand("")]
        public float maxSpeed = 2f;

        public float speed = 2f;

        public Color uniqueDebugColor;

        public Action onFinalDeath = delegate() { };

        public Brain brain;

        public bool dangerous;

        private FrameCache<float> _eRatio;

        public float enemyDist;

        public Vector3 enemyDir;

        public Data enemyData;

        public float orderDist;

        public Vector3 orderDir;

        public LazyBeats beats;

        public Action animationAction;

        private Agent.Personality personality;

        private Agent.ScoutState scoutState;

        private Vector3 scoutTarget;

        private float scoutTimer;

        private float scoutSessionEnd;

        private float nextScoutRoll;

        private float strafeSeed;

        private float previousOrderDist;

        [Serializable]
        private struct Look
        {
            public Look(Vector3 dir, float speed, float threshold)
            {
                this.dir = dir;
                this.speed = speed;
                this.threshold = threshold;
            }

            public void Set(Vector3 dir, float speed, float threshold)
            {
                this.dir = dir;
                this.speed = speed;
                this.threshold = threshold;
            }

            public Vector3 dir;

            public float speed;

            public float threshold;
        }

        private struct Personality
        {
            public float speed;

            public float reflex;

            public float boldness;

            public float wanderlust;

            public float restlessness;

            public float phase;
        }

        private enum ScoutState
        {
            Following,
            Moving,
            Paused
        }

        public delegate void MeleeBrainDelegate(Brain attacker);

        public delegate void SelectedDelegate(Agent agent, bool selected, bool showGodray);

        public interface IPostAttack
        {
            void PostAttack(Attack attack, float addedVelocity, ref bool keepGoing);
        }

        public interface IUpdateVisuals
        {
            void UpdateVisuals();
        }
    }
}
```
